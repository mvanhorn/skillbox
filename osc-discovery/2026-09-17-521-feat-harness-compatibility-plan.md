---
generated_by: osc-newfeature
bulk_run: true
type: feat
repo: kitze/skillbox
plan_id: 521
plan_date: 2026-09-17
title: "feat: harness-aware skill catalogs using Agent Skills compatibility"
feature_complexity: M
category: 1
deep_rubric_score: 8
deep_discovery_report: osc-discovery/2026-09-17-deep-report.md
process_level: PR_WELCOME
proposal_lane: false
dogfooded_general: false
dogfooded_feature: false
video_required: false
video_policy: screenshot_reel
merge_confidence: 0.79
autonomous_check_inputs:
  pre_plan_score: 8
  post_plan_score: 8
  PROCESS_LEVEL: PR_WELCOME
  NO_AI_OPPOSED: true
  NO_COMPETING_PRS: true
  HAS_PRECEDENT: false
Files:
  - path: src/server/compatibility.ts
    role: implementation
  - path: src/server/library.ts
    role: implementation
  - path: src/server/schema.ts
    role: implementation
  - path: src/server/db.ts
    role: implementation
  - path: src/server/mcp.ts
    role: wiring
  - path: src/server/access.ts
    role: implementation
  - path: src/server/app.ts
    role: wiring
  - path: src/shared.ts
    role: implementation
  - path: src/client/main.tsx
    role: wiring
  - path: src/client/access-pages.tsx
    role: wiring
  - path: src/client/skill-metrics.tsx
    role: wiring
  - path: bootstrap/SKILL.md
    role: wiring
  - path: README.md
    role: wiring
  - path: tests/compatibility.test.ts
    role: test
  - path: tests/library.test.ts
    role: test
  - path: tests/recommendations.test.ts
    role: test
---

# feat: harness-aware catalogs from Agent Skills `compatibility`

## Summary

Skillbox already **collects** client harness identity (`X-Skillbox-Harness`, MCP `initialize` `clientInfo.name`, optional `X-Skillbox-Model`) into `AccessContext` and `events`. It never **uses** that identity to decide which skills to inventory or send to Jev.

Meanwhile the Agent Skills spec defines optional `compatibility` (intended product / environment, max 500 characters). Claude Code, Cursor, and Codex share `SKILL.md` but **not** vendor frontmatter (`allowed-tools`, `hooks`, `context: fork`, `agents/openai.yaml`, Cursor `paths`). A library that grants the same bundle to mixed agents currently offers Claude-only workflows to Cursor/Codex (and the reverse), wasting context and poisoning tool choice.

This plan filters **agent-facing** discovery (MCP `search_skills` / `recommend_skills`, CLI `list`/`search`/`recommend`, and SEP listings if present) when a harness is known, using structured opt-in metadata first and `compatibility` text second. Owner UI still shows the full library. Skills with empty compatibility stay visible to every harness.

## Why this, why now

- **Dead plumbing:** `src/server/auth.ts` `requestContext` and `cli/skillbox.mjs` already send harness headers. `library.search` / `recommendationCatalog` ignore them. MCP `load_skill` accepts `context.harness` only for event recording.
- **Spec gap:** `library.metadata` preserves unknown frontmatter but does not index `compatibility`. Description may be 3000 characters; the Agent Skills cap is 1024 — out of this plan except a non-blocking owner warning if cheap.
- **Real split in the ecosystem:** Cursor docs load `.claude/skills` and `.codex/skills` for compatibility, but Claude-specific fields do not port. Public guides (e.g. Agent Skills 101) tell authors to pick a portability tier. A self-hosted library that serves **multiple** harnesses from one grant list needs a server-side filter; filesystem hosts get an implicit per-directory split Skillbox does not have.
- **Maintainer fit:** No new telemetry vendor, no execution of skill code, no multi-tenant RBAC. Filtering happens before Jev so task text + **incompatible** descriptions never leave the box. Matches `SECURITY.md` (Jev sends authorized active descriptions only).

## Non-goals

- Do not invent a new permission system or per-harness client keys (profiles/grants stay the access control).
- Do not auto-rewrite skill bodies or strip vendor frontmatter.
- Do not scrape or fingerprint models beyond the client-supplied harness string.
- Do not fail closed when harness is **unknown**: existing agents that omit headers must keep today's full granted catalog.
- Do not add incoming OAuth or hosted account identity (`docs/open-source-readiness.md` calls that a separate project).
- Do not build a general prompt-injection scanner.

## Compatibility model

### Structured (preferred)

Honor optional frontmatter:

```yaml
metadata:
  skillbox:
    harnesses: [cursor, claude-code, codex]
```

If `metadata.skillbox.harnesses` is a non-empty list of strings, it is the allow-list. Unknown keys under `metadata` remain stored (Skillbox already keeps `frontmatter` verbatim).

### Textual fallback

If no structured list is present, parse Agent Skills `compatibility` (string, trim, max 500). Classify:

| Class | Rule | Agent-facing result when harness is known |
| --- | --- | --- |
| Unspecified | missing / empty | Include |
| Environment-only | mentions packages/runtimes (python, docker, git, network) and **no** known product token | Include (not a harness constraint) |
| Product-scoped | contains at least one known product token | Include only if the client harness aliases that token |
| Structured list | `metadata.skillbox.harnesses` | Include only on list match |

### Harness aliases (closed table in `src/server/compatibility.ts`)

Normalize `clientInfo.name` / `X-Skillbox-Harness` with lowercase, strip version suffixes (`cursor/1.2` → `cursor`):

| Product tokens in `compatibility` | Aliases that match |
| --- | --- |
| `claude code`, `claude-code`, `claude` | `claude-code`, `claude`, `claude-desktop`, `anthropic` |
| `cursor` | `cursor` |
| `codex`, `openai` | `codex`, `openai`, `chatgpt` |
| `vscode`, `github copilot`, `copilot` | `vscode`, `copilot` |
| `gemini`, `gemini-cli` | `gemini`, `gemini-cli` |
| `windsurf` | `windsurf` |

Do not substring-match inside unrelated words. Tokenize compatibility on commas, slashes, and ` and `. Keep the table data-driven so tests can enumerate it.

### When to filter

Filter **only** when `principal.context.harness` is a non-empty string after normalize. Owner session (`source: web`) never filters library browse.

Agent-facing surfaces:

- `library.search` used by MCP `search_skills` and CLI list/search
- `library.recommendationCatalog` / `recommendSkills` (so Jev never sees excluded leaves)
- SEP `skills/list` if plan 520 lands; if not, this plan still stands on tools

Response extras (backward compatible JSON additions):

```json
{
  "items": ["…"],
  "compatibility": {
    "harness": "cursor",
    "filtered": true,
    "skipped": 3
  }
}
```

Old clients ignore unknown fields. `skipped` is a count only — do not leak excluded skill ids to a scoped client that wasn't granted them; leaking **granted but incompatible** ids is OK and useful for debugging (include `skippedIds` only on admin principals).

`load_skill` / `read_skill_file` / `fetch` of an explicitly requested id **remain allowed** if the client is granted the skill. Filtering is discovery, not a second ACL. Document that agents should not load skipped skills unless the user names them.

## Implementation

### 1. `src/server/compatibility.ts` (new)

- `normalizeHarness(raw: string | undefined): string | null`
- `declaredHarnesses(frontmatter: Record<string, unknown>): { mode: "any" | "products"; products: string[] }`
- `visibleToHarness(frontmatter, harness: string | null): boolean`
- `summarizeCompatibility(frontmatter): { label: string | null; products: string[] }` for UI badges

No I/O. Exhaustive unit tests here.

### 2. Persist a denormalized column

On publish (`library.metadata` / `publish`):

- Validate `compatibility` if present: string, 1–500 chars (Agent Skills). Invalid type → 400; do not guess.
- Validate `metadata.skillbox.harnesses` if present: array of 1–20 strings matching `^[a-z0-9][a-z0-9.-]{0,39}$`.
- Store `skills.compatibility` as text (raw compatibility string or empty) and `skills.harness_policy` as jsonb `{ "mode": "any"|"products", "products": [] }` so search does not parse YAML on every list.

`src/server/db.ts`: `ALTER TABLE skills ADD COLUMN IF NOT EXISTS compatibility text NOT NULL DEFAULT ''` and `harness_policy jsonb NOT NULL DEFAULT '{"mode":"any","products":[]}'`. Existing rows stay `any` (full visibility) until republish — acceptable for a new empty-by-default product.

`SkillSummary` / `SkillMetadata` in `src/shared.ts` gain `compatibility?: string` and `harnessPolicy`.

### 3. Search and recommend

`library.search`: when `p.context.harness` is set and the caller is not admin-web, add `harness_policy mode=any OR products contains alias`. Keep full-text query unchanged.

`recommendationCatalog`: same SQL filter **before** the 200 / 120k caps so mixed libraries cannot spend the Jev budget on impossible skills.

`library.record` already stores harness on events; no schema change required there.

### 4. Profiles (optional default)

Add optional `profiles.default_harness` (nullable text). MCP/CLI still prefer the live header. Default applies only when the header is absent. `access.saveProfile` + Clients/Profiles UI: a single optional text field with datalist of known aliases. Empty = current behavior.

### 5. Owner UI

Library cards (`src/client/main.tsx` / `src/client/skill-metrics.tsx`): a muted badge when `harnessPolicy.mode === "products"` (e.g. `Cursor · Claude Code`). No badge for `any`. Profiles page shows the optional default harness. Do not hide incompatible skills from the owner.

### 6. Bootstrap / README

- `bootstrap/SKILL.md`: tell agents to pass truthful harness/model context they already know; never guess (sentence already exists — extend it: discovery may omit skills marked incompatible with that harness; load remains possible if the user names the skill).
- README **Agents and CLI**: one paragraph on compatibility filtering, structured `metadata.skillbox.harnesses`, and fail-open when harness is omitted.

## Tests

`tests/compatibility.test.ts`:

- alias tables (cursor vs `claude-code/1.2`)
- environment-only `compatibility: Requires git and docker` is visible to every harness
- `Designed for Claude Code` hidden from `cursor`, visible to `claude-code`
- structured list overrides a contradictory compatibility string
- empty harness → `visibleToHarness` true for all policies

`tests/library.test.ts`:

- reader MCP `search_skills` with `X-Skillbox-Harness: cursor` omits a Claude-only fixture still granted on the profile
- same client without header still sees it
- `load_skill` of the Claude-only id still succeeds
- `recommendSkills` catalog excludes it when harness is cursor (mock evaluator; do not call TypeSafe/Gateway)

`tests/recommendations.test.ts`: catalog size / 120k path still holds on a filtered catalog.

No live provider keys. Isolated Compose Postgres via `bash scripts/test-isolated.sh`.

## Risks

- Over-filtering on noisy `compatibility` prose (`"works in any coding agent similar to Claude Code"`). Mitigation: product-token table + structured metadata as source of truth; environment-only class; fail-open without harness.
- ClientInfo names are messy (`Claude Code`, `cursor-ide`, `Codex CLI`). Alias table plus tests; unknown harness with a product-scoped skill → treat as **no match** only when a harness **is** present and normalized to a known family; if normalize yields an unknown token, fail open (do not hide the library from a new host).
- Description-length / name-length spec mismatches are **not** fixed here (quick-fix bucket).

## Merge confidence (provisional)

| Axis | Score | Note |
| --- | --- | --- |
| Process fit | 0.85 | PR_WELCOME; small, documented behavior change |
| Product strength | 0.80 | Uses existing harness plumbing; mixed-agent libraries are the point of Skillbox |
| Novelty | 0.75 | Not a Skillhub clone; filesystem hosts do not need this |
| Feasibility | 0.85 | Parse + SQL filter + badges; no new services |
| Linux+Bun testability | 0.90 | Pure unit tests + existing MCP/HTTP fixtures |
| Evidence quality | 0.60 | Day-0 repo; strong spec/docs evidence, weak in-repo issue evidence |
| **Overall** | **0.79** | |

Dogfooding skipped at discovery. Build-time evidence: two fixtures (Claude-only vs unspecified) searched under `X-Skillbox-Harness: cursor` in isolated tests.
