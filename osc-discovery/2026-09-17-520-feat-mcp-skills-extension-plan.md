---
generated_by: osc-newfeature
bulk_run: true
type: feat
repo: kitze/skillbox
plan_id: 520
plan_date: 2026-09-17
title: "feat: serve Agent Skills over MCP via SEP-2640 skills/list, skills/get, and resources"
feature_complexity: M
category: 1
deep_rubric_score: 9
deep_discovery_report: osc-discovery/2026-09-17-deep-report.md
process_level: PR_WELCOME
proposal_lane: false
dogfooded_general: false
dogfooded_feature: false
video_required: false
video_policy: screenshot_reel
merge_confidence: 0.74
autonomous_check_inputs:
  pre_plan_score: 9
  post_plan_score: 9
  PROCESS_LEVEL: PR_WELCOME
  NO_AI_OPPOSED: true
  NO_COMPETING_PRS: true
  HAS_PRECEDENT: false
Files:
  - path: src/server/skills-mcp.ts
    role: implementation
  - path: src/server/mcp.ts
    role: wiring
  - path: src/server/library.ts
    role: implementation
  - path: src/skill-references.ts
    role: implementation
  - path: src/shared.ts
    role: implementation
  - path: bootstrap/SKILL.md
    role: wiring
  - path: README.md
    role: wiring
  - path: tests/mcp-skills-extension.test.ts
    role: test
  - path: tests/skill-references.test.ts
    role: test
  - path: tests/library.test.ts
    role: test
---

# feat: SEP-2640 Skills Over MCP (revision-pinned, grant-scoped)

## Summary

Skillbox already stores Agent Skills as checksummed, revisioned packages and already speaks HTTP MCP — but only as **tools** (`search_skills`, `load_skill`, `read_skill_file`, …). Hosts that implement the Skills Over MCP extension (SEP-2640 v1) expect `capabilities.extensions["io.modelcontextprotocol/skills"]`, `skills/list`, `skills/get`, and `resources/read` of `skill://<name>/…` files with per-file `sha256:` digests.

This plan adds that protocol surface **additively**. Existing tools and `bootstrap/SKILL.md`'s mandatory unqueried `search_skills` stay. Skillbox does not execute skill code. Grants, disabled/archived lifecycle, and immutable revisions continue to gate every read.

## Why this, why now

- **Product hole:** `createMcp` in `src/server/mcp.ts` registers tools only. There is no `registerResource`, no `skills/list` / `skills/get`, and initialize does not declare `io.modelcontextprotocol/skills`.
- **Industry path:** SEP-2640 v1 (2026-07-16 scope-down) replaced the retired `skill://index.json` draft with `skills/list` + `skills/get`. OpenAI shipped Skills Over MCP while the SEP is still a draft; GitHub's MCP server has a v1 demo (`github/github-mcp-server#3046`). Arcade's explainer is the public narrative.
- **Skillbox advantage vs directory servers:** mudler/skillserver and similar hosts map a folder to MCP resources. Skillbox already has scoped clients, immutable revisions, package checksums, and `skill://<uuid>` markdown links. The unique contribution is a **grant-scoped, revision-pinned** SEP binding, not another loose file server.
- **Maintainer fit:** `SECURITY.md` and the README forbid executing uploaded skill code and treat skill bodies as untrusted guidance. Serving bytes over `resources/read` with digest verification matches that contract. Multi-tenant SaaS, incoming client OAuth, and hosted catalogs stay out of scope (`docs/open-source-readiness.md`).

## Non-goals

- Do not implement the retired `skill://index.json` / `mcp-resource-template` draft.
- Do not expose skill scripts as callable MCP tools or run them.
- Do not replace `search_skills` / `load_skill` / `read_skill_file`.
- Do not change markdown `skill://<uuid>` identity (`tests/skill-references.test.ts` requires UUIDs, not slugs).
- Do not add public skill marketplaces, anonymous catalogs, or tenant isolation.
- Do not raise Skillbox package limits to SEP's 512 files / 16 MB; keep 400 files / 8 MB.

## Protocol pin

Implement **SEP-2640 v1** as documented in `modelcontextprotocol/experimental-ext-skills` (canonical text on `modelcontextprotocol/modelcontextprotocol#2640`), specifically:

| Surface | Behavior |
| --- | --- |
| `initialize` capabilities | `extensions: { "io.modelcontextprotocol/skills": {} }` (no `directoryRead` in v1 of this PR) |
| `skills/list` | Paginated authorized **leaf** skills (bundles stay owner-side grants; listing is flat leaves, matching today's default `search_skills`) |
| `skills/get` | One skill entry by `SKILL.md` URI, including skills omitted from a page |
| `resources/read` | File bytes for URIs in that skill's `resources` set |
| Digests | `sha256:` + 64 lowercase hex of raw file bytes (already stored on `SkillFile.sha256`) |
| Frontmatter | Verbatim YAML→JSON of the pinned revision's `SKILL.md` header |

`directoryRead` / `resources/directory/read` is explicitly **out** of this change (optional SEP setting; keeps complexity at M).

## URI model

Canonical SEP URI:

```
skill://<skillId>/SKILL.md
skill://<skillId>/<relative-file-path>
```

`<skillId>` is the Skillbox skill id. `library.metadata` already requires frontmatter `name` to match the id when present, so the SEP "final segment equals `name`" rule holds for well-formed skills.

**Alias (Skillbox-native, not markdown):** `skills/get` and `resources/read` also accept `skill://<referenceId>/SKILL.md` and `skill://<referenceId>/<path>`, resolving through existing `resolveReferenceId`. Responses always echo the **canonical slug URI**. Markdown extraction in `extractSkillReferences` stays UUID-only.

**Revision:** listings are point-in-time snapshots of the **current** granted revision (same as `load_skill` without `revision`). Put Skillbox extras on `_meta` with the required prefix, not invented top-level fields:

```json
{
  "io.modelcontextprotocol.skills/revision": "<revision-id>",
  "io.modelcontextprotocol.skills/referenceId": "<uuid>",
  "io.modelcontextprotocol.skills/checksum": "<package-checksum>"
}
```

Historical revisions remain available through existing `load_skill({revision})` / HTTP, not through SEP listing (SEP has no revision query). Document that hosts that need a frozen tree should `skillbox fetch id@REVISION`.

## Authorization and safety

- Same `Principal` as tools. Unauthorized / disabled / archived skills: `skills/get` and `resources/read` return JSON-RPC `-32602` (Invalid params), matching SEP and Skillbox's existing "not found" (do not leak 403 vs 404).
- `skills/list` is grant-filtered like `library.search` default (`kind: skill`, not archived, not disabled).
- Re-check grants on every `skills/get` / `resources/read` (profiles apply immediately; see existing library tests).
- Never return file bytes for a path absent from the entry's `resources` set.
- Binary files: return MCP resource blobs (base64) with a sensible `mimeType`; do not pretend they are UTF-8 text. Existing `read_skill_file` 160 KB / NUL-byte limits apply only to the **tool**; SEP `resources/read` may return any stored file up to Skillbox's 2 MB per-file cap.
- Server `instructions` may mention that SEP hosts can use `skills/list`; they must still say tools remain valid and `search_skills` is the Skillbox bootstrap inventory step.

## Implementation

### 1. `src/server/skills-mcp.ts` (new)

Pure helpers, easy to unit-test without spinning MCP:

- `parseSkillResourceUri(uri) → { idOrReference, path } | null`
- `canonicalSkillUri(skillId, path)`
- `skillEntry(principal, skill, revision) → { uri, frontmatter, resources, _meta }`
- `listSkillEntries(principal, { cursor, limit })` using `library.search` + `revisionFor`
- `getSkillEntry(principal, uri)`
- `readSkillResource(principal, uri) → { mimeType, bytes, digest }`

Cursor pagination: opaque `offset:<n>` is enough (catalogs cap at 500 browse / 200 recommend today). Keep an entry atomic (never split one skill's `resources` across pages).

### 2. `src/server/mcp.ts` (wiring)

After constructing `McpServer`:

1. Declare the extension on the underlying SDK `Server` initialize result. Inspect `@modelcontextprotocol/sdk@1.30` (`bun.lock` pins 1.30.0) for `capabilities.extensions`. If the SDK has no first-class extensions bag, merge via the documented server capability setter rather than forking the SDK.
2. Register `skills/list` and `skills/get` as custom JSON-RPC methods (`server.server.setRequestHandler` or SDK equivalent). Do **not** fake them as tools.
3. Register resource templates for `skill://{id}/{+path}` so `resources/list` / `resources/read` work for hosts that only speak Resources. Listing may be partial; `skills/list` is authoritative enumeration.
4. Keep `handleMcp`'s existing `initialize` → `library.record(..., "connect", { harness })` behavior.

The stdio CLI (`cli/skillbox.mjs mcp`) already forwards raw JSON-RPC to `POST /mcp`. No CLI protocol change is required if HTTP handles the new methods.

### 3. `src/skill-references.ts`

Add `parseSkillResourceUri` **next to** `referenceId`, without weakening the UUID-only markdown extractor. Tests must keep `referenceId("skill://android-engineering") === null`.

### 4. Docs (product-facing, no process branding)

- `README.md` **Agents and CLI**: one short paragraph that SEP-capable hosts may call `skills/list` / `skills/get` / `resources/read`, that listings are grant-scoped current revisions, and that `search_skills` remains the bootstrap inventory step.
- `bootstrap/SKILL.md`: optional sentence for hosts that speak the extension; do not make SEP methods mandatory.

## Tests (`tests/mcp-skills-extension.test.ts`)

Follow the existing `/mcp` JSON-RPC style in `tests/library.test.ts` (Bearer client, `Accept: application/json, text/event-stream`).

1. **Initialize** advertises `io.modelcontextprotocol/skills` and still lists the same tool counts (reader 5 / admin 9).
2. **List** returns only granted leaves; disabled and unauthorized ids are absent; each entry has verbatim `frontmatter.name`/`description`, complete `resources` with `sha256:` digests matching stored file hashes, and canonical `skill://<id>/SKILL.md`.
3. **Get** by slug URI and by `referenceId` alias returns the same canonical URI.
4. **Get / read** of another client's skill is `-32602`.
5. **resources/read** of `SKILL.md` bytes equal the published file; digest matches the list entry; a path not in `resources` fails.
6. **Publish a new revision** changes digests; a subsequent `skills/get` returns the new set (point-in-time).
7. **Tools still work:** `tools/call` `search_skills` / `load_skill` unchanged on the same server instance.
8. **No execution:** fetching a skill that contains `scripts/run.sh` with `executable: true` only returns bytes.

Run with the isolated Compose path the README specifies (`bash scripts/test-isolated.sh`) plus `bun run typecheck`. Linux + Bun only; no paid provider keys.

## Risks

- SEP-2640 is still a draft. Pin comments in code to the v1 method names (`skills/list`, `skills/get`) and keep tools as the stable Skillbox API if the SEP renames again.
- SDK 1.30 may not expose `capabilities.extensions` cleanly. Fail the PR if initialize cannot advertise the extension without a brittle monkey-patch; do not ship a silent tools-only fallback labeled as SEP support.
- Bundle grants: listing must expand bundles to leaves (already `library.search` default). Loading a bundle via SEP URI is not required.

## Merge confidence (provisional)

| Axis | Score | Note |
| --- | --- | --- |
| Process fit | 0.85 | PR_WELCOME, MIT, no propose-first, issues enabled |
| Product strength | 0.90 | Turns Skillbox into a protocol-native skills host |
| Novelty | 0.90 | No Skillbox issue/PR covers this; tools-only today |
| Feasibility | 0.65 | Draft SEP + SDK extension plumbing |
| Linux+Bun testability | 0.90 | Existing `/mcp` JSON-RPC tests |
| Evidence quality | 0.55 | Day-0 repo; Path B social; strong protocol docs |
| **Overall** | **0.74** | |

Dogfooding is skipped at discovery (`dogfooded_general: false`). Build-time U6 should exercise initialize → `skills/list` → `resources/read` against the isolated test database, not a personal library.
