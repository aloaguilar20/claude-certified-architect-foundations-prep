# CCAR-F Cheat Sheet — one page per domain

Last-week consolidation. Details and citations live in the [chapters](../guide/).

---

## D1 — Agentic Architecture & Orchestration (27%)

- **API is stateless** → resend full history every call.
- **Loop control = `stop_reason`**: `tool_use` → execute & continue; `end_turn` → done. Never text-matching, never iteration caps as primary stop.
- Tool results → `tool_result` block in a **user** message (no "tool" role).
- Context window failures: **lost-in-the-middle** (put key info at start/end), tool-result accumulation (trim), lossy summarization (numbers → "roughly").
- Subagents: **isolated context, nothing inherited**, pass everything in the prompt; coordinator needs `Task` in allowed tools; parallel spawning = multiple Tasks in one response.
- **Hooks vs prompts:** financial/legal/safety consequence → hook (deterministic). Miscalibrated judgment → explicit criteria + examples, *not* a hook. Always the **least complex fix**.
- Decomposition: predictable → fixed pipeline; open-ended → dynamic adaptive. 10+ files in one pass → **attention dilution** → per-file passes + integration pass.
- Sessions: resume = same thread (stale-file risk); fork = branch from shared context; degraded/stale → fresh session with a summary.

## D2 — Tool Design & MCP (18%)

- **The description IS the selection mechanism.** Overlapping/vague descriptions → wrong tool. Fix descriptions before adding routers/classifiers.
- Good description: what it does/returns, input formats + examples, edge cases, **when to use vs alternatives**.
- Agent prefers built-in over MCP tool → strengthen the MCP description (unique data/advantages).
- `tool_choice`: `auto` (model decides) · `any` (must call *some* tool → guaranteed structure, flexible pick) · `tool` (forced specific first step) · `none` (text only).
- MCP primitives: **Tools** (act) · **Resources** (read context — an instant "map", no exploratory calls) · **Prompts** (templates).
- Config: `.mcp.json` at repo root = team, in VCS, secrets via `${ENV_VAR}`; user scope = personal experiments. Standard integrations → community servers; custom only for unique workflows.
- Errors: `isError: true` + **category, retryability, message, attempted query, partial results** — never "Operation failed".

## D3 — Claude Code (20%)

- CLAUDE.md scopes: managed → user (`~/.claude/CLAUDE.md`, not shared) → project (`./CLAUDE.md` or `.claude/CLAUDE.md`, in VCS) → local → directory (on-demand).
- **Trap:** teammate missing conventions = they were user-level; belong project-level in VCS.
- Imports: `@path` (no space), relative to the *importing file*, max **four hops** (older material says five).
- `.claude/rules/` + `paths:` globs → loads only for matching files. Scattered file types → path rule; one directory → directory CLAUDE.md; procedure → skill; always-true fact → CLAUDE.md.
- Skills = commands (merged). Frontmatter: `context: fork` (isolated subagent), `allowed-tools`, `argument-hint`, `disable-model-invocation`.
- **Plan mode** for large/ambiguous/unfamiliar/architectural; direct execution for small clear fixes; combined = plan → approve → execute. Explore subagent isolates verbose exploration.
- `/compact` frees context (risk: loses exact numbers); `/memory` edits persistent memory files.
- Tools: Glob (names) · Grep (contents) · Read · Write · Edit (unique match; fallback Read→modify→Write) · Bash. Investigate incrementally.
- CI/CD: `claude -p` (headless) + `--output-format json` + `--json-schema`; **separate instance for review** (a session won't challenge its own code); re-reviews get prior findings + "only new/unresolved".

## D4 — Prompt Engineering & Structured Output (20%)

- Schema-backed tool use guarantees **syntax, never semantics**.
- `required` only for always-present data (else fabrication); nullable types for maybe-absent; enums get `other` + detail and `unclear`.
- Inconsistent behavior across attempts → **add 2–4 examples**, not more prose. Examples define boundaries (flag this / not that), formats, informal units.
- Normalization rules in the prompt: ISO dates, currency codes, decimal fractions.
- False positives → explicit include/exclude criteria + severity definitions with examples.
- Chaining for predictable multi-step; interview pattern (ask first) for unfamiliar domains; test-driven iteration; interacting fixes → one batch, independent fixes → sequential.
- **Retry-with-feedback** fixes format/structure/arithmetic (info present). It **cannot** fix absent information — recognize which side a failure is on.
- Pydantic: structural + business validation, error messages for retry loops, generates the schema (single source of truth).
- Self-correction: `stated_total` vs `calculated_total` + `conflict_detected`; `detected_pattern` for false-positive analysis.

## D5 — Context Management & Reliability (15%)

- Error taxonomy: transient (retry w/ backoff) · validation (fix input) · business (explain, alternative) · permission (escalate).
- Anti-patterns: generic errors, silent suppression (empty ≠ failure!), full abort on one failure (keep partials + annotate coverage), infinite subagent retries (1–2 local, then propagate).
- Escalate on: explicit human request (immediately), policy silence, no progress, financial threshold (hook), ambiguous identity (ask, don't guess).
- **Not** triggers: sentiment, self-rated confidence, bolt-on classifiers. Frustration ≠ "get me a manager": acknowledge → resolve → escalate on reiteration.
- Handoff = self-contained structured summary (human doesn't see the transcript).
- Calibration: field-level confidence + labeled validation sets; stratified sampling — aggregate 97% can hide 40% errors in one document type.
- Context: fact blocks in every prompt; trim tool results (PostToolUse); position-aware layout; scratchpad files; subagent delegation with minimal context budgets; state manifests for crash recovery.
- Provenance: claim + source + date + confidence as a unit; keep conflicting values *both*, with methodology + dates; render by content type.
- Batches: **50% cost, up to 24h, `custom_id`**, single invocation per request (no in-batch agent loop). Human waiting → sync; overnight/bulk → batch. Deadline math: submit ≥24h before deadline.

---

## The 5-step scenario method

Outcome → binding constraint (*must, first, shared, without interaction, policy is silent*) → failure mode → **least complex fix** → test every distractor. Never leave a blank.
