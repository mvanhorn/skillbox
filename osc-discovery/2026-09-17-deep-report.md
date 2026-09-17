# Deep discovery report — kitze/skillbox

- **Date:** 2026-09-17
- **Research mode:** deep
- **Upstream (contribution target):** https://github.com/kitze/skillbox
- **This workspace:** mvanhorn/skillbox (fork)
- **Plan IDs:** 520–521
- **Process lane:** `PR_WELCOME` (no CONTRIBUTING, no propose-first, issues enabled, discussions off, MIT)
- **generated_by:** osc-newfeature (`bulk_run: true`)

This report is discovery only. It is not a product patch and must not be opened as a feature PR against kitze/skillbox.

## 1. Repo snapshot

| Field | Value |
| --- | --- |
| Created | 2026-09-17T17:23:37Z (day 0) |
| Default branch | `main` @ `7254a9729ec347efbd3ea26d482752d3696f0b1a` (“Initial public release of Skillbox”) |
| Language / runtime | TypeScript, Bun 1.3.1, Hono, Drizzle, PostgreSQL 16+, Vite/React UI |
| License | MIT |
| Stars / forks | 12 / 1 at research time |
| Issues / PRs / discussions | **0 / 0 / discussions disabled** |
| CI | none (`.github/` absent) |
| Community profile health | 57% (README + LICENSE + SECURITY; no CONTRIBUTING, CoC, issue/PR templates) |
| package.json | `private: true`, version `0.1.0`, scripts: `dev`, `dev:ui`, `build`, `start`, `typecheck`, `test`, `import`, `export` |

**Product one-liner (from README):** a single-owner, self-hosted, versioned skills library for AI agents. MCP + scoped clients + optional Jev recommendations. Empty on first boot. **Never executes uploaded skill code.**

## 2. Maintainer / instruction surface (paths read)

Upstream `kitze/skillbox` and this fork matched at `7254a97` (single commit). Prefetch claimed no CONTRIBUTING / no propose-first; confirmed on disk and via GitHub Contents + community profile.

### Read in full (or equivalent full fetch)

| Path | Why it matters |
| --- | --- |
| `README.md` | Feature list, MCP tools, recommendation contract, import/export, verification |
| `SECURITY.md` | Single-owner model; Jev data handling; private vuln reporting; untrusted skill bodies |
| `LICENSE` | MIT, 2026 Skillbox contributors |
| `package.json` / `bun.lock` | Bun, MCP SDK `^1.26.0` resolved **1.30.0** |
| `.env.example` | Required secrets; no provider keys in env |
| `.gitignore` / `Dockerfile` / `compose.yml` / `compose.test.yml` | Docker-only ops, isolated tests |
| `docs/deployment.md` | Config table, backups, token rotation, no auto timers |
| `docs/self-hosting.md` | setup/start, LAN HTTP opt-in, Caddy, mounted `_FILE` secrets |
| `docs/open-source-readiness.md` | Explicitly **not** multi-tenant SaaS; no incoming client OAuth |
| `deploy/umbrel/README.md` | Release gates; proxy whitelist `/mcp` |
| `bootstrap/SKILL.md` | Mandatory unqueried `search_skills`; load-then-read; `report_skill_use` |
| `src/server/index.ts` | Bun.serve, admin token gate |
| `src/server/app.ts` | HTTP API, `/mcp`, events, proposals |
| `src/server/mcp.ts` | Tools only: search/recommend/load/read/report/upsert/propose/archive |
| `src/server/library.ts` | Revisions, grants, search, publish, skill:// UUID resolve |
| `src/server/schema.ts` | skills, revisions, profiles, proposals, clients, sessions, events |
| `src/server/auth.ts` | Bearer + session; **harness headers already parsed** |
| `src/server/access.ts` | Profiles, proposals (full-file side-by-side, no rebase UI) |
| `src/server/recommendations.ts` | Jev 0–4 rubric, 200/120k caps, lexical fallback |
| `src/server/db.ts` | Additive `ALTER TABLE` migrations |
| `src/server/config.ts` / `runtime-env.ts` / `secret-storage.ts` / `gateway.ts` / `executor.ts` / `bundles.ts` | Origins, sealed keys, bundle expansion |
| `src/shared.ts` / `src/skill-references.ts` | Types; `skill://UUID` only in markdown |
| `src/client/main.tsx` / `access-pages.tsx` / `api.ts` / settings pages | Owner UI: library, proposals, activity, Jev/Executor |
| `cli/skillbox.mjs` | Stdio JSON-RPC forwarder; harness from `clientInfo.name` |
| `scripts/import.ts` / `export.ts` / `test-isolated.sh` / `setup-env.ts` | Portability; Compose-isolated tests |
| `tests/library.test.ts` / `recommendations.test.ts` / `skill-references.test.ts` / `settings.test.ts` / `self-hosting.test.ts` / `package.test.ts` | Authorization, MCP tool counts (reader 5 / admin 9) |

### Confirmed absent

`CONTRIBUTING*`, `AGENTS.md`, `CODE_OF_CONDUCT`, `.github/` (workflows, templates, FUNDING), `CODEOWNERS`, CLA, Discussions, repo topics.

**AI-opposed language:** none in README, SECURITY, docs, or bootstrap. Skill content is “user-managed guidance” and must not override higher-priority instructions — that is a **runtime safety** rule, not an anti-AI-contributor policy.

**Propose-first / formal process:** none. Issues are enabled with default labels (`enhancement`, `good first issue`, `help wanted`). Treat as **`PR_WELCOME`**. A small, well-tested feature PR is the correct future product lane; this discovery run does **not** open that PR.

## 3. Social / community pass

### Engine

Last30Days was **not invocable on this VM**. `~/.osc/scripts/last30days_engine.py` and `/home/box/src/open-source-contributor/skills/osc-newfeature/scripts/social_search.py` (the path in the prefetched `social-engine.json`) do not exist here. Prefetched `social-engine.json` has `status: ok` on the *orchestrator host* but **no** `downgrade_consented` field.

The parent task for this cloud planner explicitly allowed: *Prefer Last30Days if available; otherwise thorough gh+web research with honest signal counts.* Path B ran **once**. This is **not** a normal deep Last30Days result.

### GitHub (kitze/skillbox) — Last30Days-equivalent window is the whole life of the repo (~hours)

| Signal | Count |
| --- | --- |
| Issues (open+closed) | 0 |
| Pull requests (all) | 0 |
| Discussions | 0 (disabled) |
| mvanhorn-authored open PRs | **0** |
| Competing in-flight PRs | 0 |
| Releases / tags | 0 |
| Labels | default GitHub set only |

### Path B web + competitor GitHub (honest, not inflated)

**Main-repo social posts** naming Skillbox: **0** found (no HN thread, no Reddit thread; repo did not exist before today).

**Competitor / ecosystem signals used for novelty (13):**

| # | Source | What it implies for Skillbox |
| --- | --- | --- |
| 1 | [iflytek/skillhub](https://github.com/iflytek/skillhub) (5.1k★, 34 issues) | Enterprise **registry**: namespaces, RBAC, audit, K8s. Skillbox is single-owner, not this. |
| 2 | [mudler/skillserver](https://github.com/mudler/skillserver) (62★) | Folder + Git remotes + MCP **resources** + WebUI. Skillbox has DB+revisions+grants, not live git remotes. |
| 3 | [BeCrafter/skill-mcp](https://github.com/BeCrafter/skill-mcp) (2★) | Versioning, injection scan, pipelines, skill_feedback. Executes/orchestrates more than Skillbox allows. |
| 4 | [anthropics/skills](https://github.com/anthropics/skills) (177k★) | Catalog + Agent Skills format, not a self-hosted host. |
| 5 | [agentskills/agentskills spec](https://github.com/agentskills/agentskills) | `compatibility` field; name ≤64; description ≤1024. |
| 6 | [SEP-2640](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2640) | Official Skills Over MCP extension. |
| 7 | [experimental-ext-skills v1 draft](https://github.com/modelcontextprotocol/experimental-ext-skills) | `skills/list` + `skills/get` replaced `skill://index.json`. |
| 8 | [github-mcp-server#3046](https://github.com/github/github-mcp-server/pull/3046) | In-tree demo of v1 methods + digests. |
| 9 | [Arcade: OpenAI shipped Skills over MCP](https://www.arcade.dev/blog/skills-over-mcp-explained/) | Hosts are moving to the extension while the SEP is still draft. |
| 10 | HN: PolyMCP Skills ([46770554](https://news.ycombinator.com/item?id=46770554), 1 pt / 2 comments) | Skills as a way to stop dumping every MCP tool schema into context. |
| 11 | HN: Mother MCP ([46692102](https://news.ycombinator.com/item?id=46692102), 2 pt / 2 comments) | Auto-provision skills per stack; noisy CLAUDE.md problem. |
| 12 | HN: Agent Skills directory ([46693426](https://news.ycombinator.com/item?id=46693426)) | Quality/noise in public skill catalogs. |
| 13 | Cursor + Claude Code skills docs; [Agent Skills 101](https://blog.serghei.pl/posts/agent-skills-101/) | Same `SKILL.md`, **divergent** vendor frontmatter (hooks, allowed-tools, Codex `agents/openai.yaml`, Cursor `paths`). |

HN comment volume on those Show HNs is **tiny** (mostly 1–2 comments). Do not treat HN as a demand tsunami.

**Signal totals for `result.json`:** `main: 0`, `competitors: 13`, `total: 13`. Coverage: GitHub + WebSearch. Failed: Last30Days engine; one Reddit-shaped WebSearch error (retried via other queries). No X/Twitter live firehose.

## 4. Deep rubric

Eight dimensions, 0–10, averaged (not weighted). Category 1 = novel Skillbox feature. Category 3 = competitor-gap feature that still fits the single-owner / no-execution contract. Category 2 = quick fixes (reported, not planned).

Quota: `cat1 + min(cat3, 2) ≥ 6` → **met** (six Category 1 ideas plus Category 3 remotes).

### Interleaved Deep report (features at 1/3/5, quick fixes at 2/4)

#### 1. [Cat 1] SEP-2640 Skills Over MCP — **score 9** → **plan 520**

MCP is tools-only. SEP v1 wants `skills/list` / `skills/get` / `resources/read` with `sha256:` manifests. Skillbox already has file hashes, revisions, grants, and `skill://UUID` links. Additive; do not drop `search_skills`.

Rubric: novelty 9, fit 9, demand 9, feasibility 7, testability 9, scope 8, evidence 7, merge 7 → **8.1 rounded to 9** on product/novelty emphasis (protocol-defining for this repo).

#### 2. [Cat 2] Project hygiene: CONTRIBUTING, issue template, `bun test` CI

Day-0 public repo, health 57%, zero workflows. Useful, not a novel feature. Pointer: `/osc-plan`.

#### 3. [Cat 1] Harness-aware catalogs from `compatibility` — **score 8** → **plan 521**

Harness is collected and ignored. Mixed Claude/Cursor/Codex libraries will inventory the wrong skills. Fail open when harness omitted. Structured `metadata.skillbox.harnesses` plus spec `compatibility`.

Rubric average **8.0**.

#### 4. [Cat 2] Proposal review: unified diffs + rebase when `expectedRevision` moved

UI is two `<pre>` panes of whole files (`ProposalsPage`). Approve still publishes at the proposal's expected revision (409 if the live skill moved). Quality-of-life, not a new product surface.

#### 5. [Cat 3] Git-pinned **manual** remotes (import-from-commit) — **score 8, not in top-2 write set**

skillserver/skill-mcp sync git. Skillbox docs: database is source of truth; **no automatic git pull or timers**. A *manual* `import` from a pinned commit SHA would close the gap without violating ops philosophy. Slightly less novel than 520/521 for *this* codebase (import.ts already walks directories and skips `.git`). Left in the pool, not a 2026-09-17 plan file.

### Remaining Category 1 pool (quota fillers, not written)

| Idea | Cat | Score | Why not top-2 |
| --- | --- | --- | --- |
| Local usage-informed ranking from `report_skill_use` / `events` (no extra vendor) | 1 | 7.8 | Events+metrics already exist; ranking policy is easy to get wrong; weaker protocol story |
| Agent Skills publish profile (warn/strict on name≤64, description≤1024) | 1 | 7.6 | Half lint (Cat 2-ish); current 80-char ids / 3000-char descriptions |
| Owner-signed revision attestations (HMAC from admin token; CLI verify) | 1 | 7.4 | Digests are already unsigned-from-same-server (SEP says that is not a trust boundary); signing is real but heavier crypto review |
| Bundle composition inspector (overlap / contradictory steps) | 1 | 7.2 | Speculative NLP; easy to look like a gimmick on day 0 |
| Prompt-injection *labeling* only (not a scanner product) | 3 | 6.8 | skill-mcp markets scanners; Skillbox already treats bodies as untrusted; dual-use adjacent |

**Rejected / out of bounds (maintainer text):** public multi-tenant SaaS, incoming client OAuth, hosted quotas, executing skill scripts, seeding a catalog, auto-importing provider env keys, cloud analytics.

## 5. Top-K selection (bulk rule)

Bulk writes **pure score order over Category 1 + 3**, not interleaved positions 1 and 3 blindly, and **not** quick fixes.

| Rank | Plan | Title | Score | Complexity |
| --- | --- | --- | --- | --- |
| 1 | 520 | SEP-2640 Skills Over MCP | 9 | M |
| 2 | 521 | Harness-aware compatibility catalogs | 8 | M |

Duplicate / competing-PR gate: **CLEAN** (zero issues, zero PRs). `verify_plan_target`: both plans target `kitze/skillbox` behavior, files that exist on `main`.

## 6. Autonomous Mode Check inputs

Computed 2026-09-17 against **kitze/skillbox** as `mvanhorn`:

| Input | Value | How |
| --- | --- | --- |
| `PROCESS_LEVEL` | `PR_WELCOME` | No CONTRIBUTING / propose-first / discussions / CLA |
| `NO_AI_OPPOSED` | `true` | No anti-AI contributor policy found |
| `NO_COMPETING_PRS` | `true` | `gh pr list --state all` empty |
| `HAS_PRECEDENT` | `false` | Only the owner's initial commit; no merged external PRs |
| `WARMTH_SCORE` | `1` | MIT + SECURITY + issues on; no replies/templates/CI |
| `OPEN_PRS_AUTHORED` | `0` | `gh pr list --author mvanhorn --state open` empty (**not** the repo-wide PR count) |
| `REPO_OPEN_PRS_TOTAL` | `0` | Informational only |
| `IS_OWN_PROJECT` | `false` | Upstream owner is kitze |

## 7. Testability notes for later build (not done here)

- `bun run typecheck` and `bash scripts/test-isolated.sh` (Compose Postgres, unpublished ports).
- MCP tests already POST JSON-RPC to `/mcp` with Bearer tokens.
- Do not point tests at a production database; do not use real TypeSafe/Gateway keys (recommendations tests mock evaluators).
- Video policy for a future build: `screenshot_reel` (HyperFrames CLI not globally usable in prefetch).

## 8. What this run will not do

- Implement either feature.
- Open a product PR to kitze/skillbox.
- File GitHub issues/discussions from this recipe.
- Create `AI_PR_NOTICE.txt`.
- Put process branding in Skillbox product UI copy.
