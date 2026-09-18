<!-- Interactive twin: ../flashcards.html (embedded CARDS array) — keep BOTH in sync when content changes. -->

# CCAR-F Flashcards

Cover the answer, quiz yourself. Grouped by domain; shuffle in practice.

## Domain 1 — Agentic Architecture

**Q: What is the only reliable signal that an agentic task is complete?**
A: `stop_reason == "end_turn"`. Text matching and iteration caps are named anti-patterns.

**Q: The model wants a tool executed. What does the response contain and what do you do?**
A: `stop_reason == "tool_use"` with a `tool_use` content block → execute the tool, append a `tool_result` block *in a user message*, call the API again with full history.

**Q: Why must you resend the whole conversation every call?**
A: The API is stateless — no state persists between requests.

**Q: What context does a subagent inherit from its coordinator?**
A: None. Context isolation is total; everything needed must be passed explicitly in the spawn prompt.

**Q: What must a coordinator's allowed tools include to spawn subagents?**
A: The `Task` tool.

**Q: How do you run three subagents in parallel?**
A: The coordinator emits multiple Task calls in a single response.

**Q: Refunds over $500 must always be blocked. Prompt or hook?**
A: PreToolUse hook — financial consequence demands deterministic enforcement.

**Q: The agent escalates the wrong cases. Hook?**
A: No — that's miscalibrated judgment. Fix with explicit criteria and few-shot examples; a hook can't encode a judgment boundary.

**Q: What is the lost-in-the-middle effect and its mitigation?**
A: Long-input details in the middle get missed; place key information near the beginning or end.

**Q: Fixed pipeline vs dynamic decomposition — when each?**
A: Fixed: predictable structure, known steps, reproducibility. Dynamic: open-ended scope, each step depends on prior results.

**Q: Why does single-pass review of 14 files fail?**
A: Attention dilution — uneven depth, inconsistent flagging, missed obvious bugs. Use per-file passes + an integration pass.

**Q: resume vs fork_session?**
A: Resume continues one specific session; fork copies history to a new independent branch (original untouched) — use to compare approaches. Stale files/degraded context → fresh session with a summary instead.

**Q: Two subagents keep re-reading the same files and duplicating findings. Fix?**
A: The coordinator partitions scope: each spawn prompt names a distinct set of files/questions and says what the other subagents cover.

**Q: Three analyses: #1 and #2 independent, #3 compares them. How do you orchestrate?**
A: Emit #1 and #2 as parallel Task calls in one response, then spawn #3 with both results passed explicitly in its prompt.

**Q: Subagents are defined correctly but the coordinator never delegates. First thing to check?**
A: Its allowed tools must include the spawning tool — `Task` in exam material, renamed `Agent` in current Claude Code (`Task` still works as an alias).

**Q: Goal-oriented vs procedural subagent instructions — which and why?**
A: Goals + quality criteria + output format. Rigid step lists stop the subagent adapting to what it finds; the coordinator keeps control through the output contract.

**Q: Do subagents talk to each other?**
A: No — hub-and-spoke: all communication flows through the coordinator for visibility and error control. (Current docs allow limited nesting depth, capped by `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`, but the exam design keeps the coordinator in charge.)

## Domain 2 — Tools & MCP

**Q: What is the primary mechanism by which Claude selects a tool?**
A: The tool's **description**.

**Q: The agent keeps using built-in Grep instead of your richer MCP search tool. Keyed fix?**
A: Strengthen the MCP tool's description — concrete advantages and unique data. Not removal, hooks, or routing layers.

**Q: tool_choice values and effects?**
A: `auto` (model decides), `any` (must call some tool), `tool`+name (must call that tool), `none` (no tools, text only).

**Q: You need guaranteed structured output but the right schema among three is unknown. Setting?**
A: `tool_choice: {"type": "any"}` — forces a tool call, model picks the best schema.

**Q: The three MCP primitives?**
A: Tools (actions), Resources (readable context), Prompts (templates).

**Q: Team-shared MCP config — file and secret handling?**
A: `.mcp.json` at the project root, in version control, secrets via environment-variable expansion like `${GITHUB_TOKEN}`.

**Q: What makes an MCP error useful to an agent?**
A: `isError: true` plus structure: category, `isRetryable`, message, attempted query, partial results — enough to choose retry / rephrase / escalate.

**Q: MCP server scopes: local vs project vs user?**
A: Local (default): private, this project only. Project: `.mcp.json`, shared via VCS. User: private, all your projects.

**Q: Shared MCP server configured, teammate sees no tools. Check first?**
A: That the `${VAR}` used in `.mcp.json` for auth is set in their environment, then verify discovery with `/mcp` or `claude mcp list`. Never commit tokens.

## Domain 3 — Claude Code

**Q: New teammate's Claude Code ignores team conventions. Likeliest cause?**
A: Conventions in someone's user-level `~/.claude/CLAUDE.md` instead of a project-level CLAUDE.md in version control.

**Q: @path import rules?**
A: `@` directly before the path; relative paths resolve relative to the importing file; recursion up to four hops (current docs — older materials say five).

**Q: Test conventions apply to test files scattered everywhere. Directory CLAUDE.md or rule?**
A: `.claude/rules/` file with `paths:` glob frontmatter (e.g. `**/*.test.ts`) — loads only when matching files are touched.

**Q: Skill vs CLAUDE.md?**
A: Skill = on-demand procedure (loads when used); CLAUDE.md = always-loaded facts/standards.

**Q: What does `context: fork` do in skill frontmatter?**
A: Runs the skill in an isolated subagent so verbose output doesn't pollute the main session.

**Q: When is plan mode the keyed choice?**
A: Large multi-file changes, several plausible approaches, architectural decisions, unfamiliar codebases, wide migrations. Small clear fixes → direct execution.

**Q: What risk comes with /compact?**
A: Summarization can lose exact numbers, dates, and specifics.

**Q: Find every call site of a function in an unfamiliar repo — first tool?**
A: Grep (content search). Glob is for file names; Read for loading specific files.

**Q: The only correct way to run Claude Code in a CI pipeline?**
A: Headless: `claude -p "..."` (with `--output-format json` and optionally `--json-schema` for machine-parseable output).

**Q: Why review code with a different instance than the one that wrote it?**
A: The generating session retains its reasoning and won't challenge its own decisions.

**Q: Glob or Grep?**
A: Glob matches file *paths* (`**/*.test.ts`); Grep searches file *contents* (identifiers, imports, error strings). Callers are found with Grep, not Glob.

**Q: Token-efficient way to explore a codebase?**
A: Narrow-then-read: Glob/Grep to locate (file-list or count output, path/glob/type filters) → Read only the relevant files or line ranges → Grep usages → repeat. Never read everything up front.

**Q: Need one function in a 6,000-line file. How?**
A: Grep with line numbers to locate it, then Read with `offset`/`limit` for just that window.

**Q: When should you use Bash instead of Grep/Glob/Read?**
A: Only for what needs a shell: git history/blame, running tests, builds. Use dedicated tools for search and reading — bounded, structured results.

**Q: Exploration is filling the main context. Fix?**
A: Delegate to the read-only Explore subagent (isolated context, returns a summary), keep findings in a scratchpad file, and use `/compact` when needed.

**Q: Three parts of a CI review configuration?**
A: (1) Project CLAUDE.md with the review standards; (2) restricted tools, e.g. `--allowedTools` read-only; (3) `--output-format json` + `--json-schema` so a script can post inline comments. Run with `claude -p`.

**Q: Prevent runaway unattended CI runs?**
A: `--max-turns`, `--max-budget-usd`, and a minimal tool allowlist. Not `--dangerously-skip-permissions`.

**Q: Generated code fails some tests. Best next prompt?**
A: Run the suite and give Claude the failing output (expected vs actual); have it fix and re-run. Concrete feedback beats "try again".

**Q: Bug report arrives. Most reliable workflow?**
A: Write a failing test that reproduces it → fix until it and the suite pass. The test verifies the fix and guards regressions.

**Q: Generated tests are trivial (`toBeDefined`). Fix?**
A: Provide existing test files, state fixture conventions to reuse, and define what makes a test meaningful (asserts behavior, covers edge cases, fails if the logic breaks).

**Q: Several review issues to relay. How to batch them?**
A: Interacting issues together in one message (one coherent fix); independent issues as separate, focused requests.

**Q: Bot says "LGTM". Auto-merge?**
A: No. Keep deterministic required checks (tests, lint, build) and human approval for risky changes as the merge gate; AI review is advisory. Never let the authoring session approve its own PR.

**Q: "Every edit must be formatted and linted, no exceptions." Where?**
A: A PostToolUse hook — deterministic. CLAUDE.md is guidance, not a guarantee.

## Domain 4 — Prompt Engineering & Structured Output

**Q: What does a JSON schema guarantee — and not?**
A: Guarantees syntax and structure (valid JSON, required fields). Does NOT guarantee semantic correctness.

**Q: Why do required fields on sometimes-missing data backfire?**
A: They push the model to fabricate values. Use nullable types so it can honestly return null.

**Q: Two schema-design escape valves for categorization?**
A: `"other"` (+ detail field) and `"unclear"` enum values.

**Q: Output format interpreted inconsistently across attempts. Most effective fix?**
A: 2–4 concrete input/output examples — they beat longer prose descriptions.

**Q: When is retry-with-feedback futile?**
A: When the information is absent from the source (or lives in a never-supplied document). Retry only fixes present-but-misorganized data.

**Q: What goes into a retry prompt after a validation failure?**
A: The original document, the previous (incorrect) extraction, and the specific error ("total=150 but line items sum to 145").

**Q: Pydantic's three exam-relevant roles?**
A: Structural validation, custom business-rule validators, and generating the JSON schema for tool_use (single source of truth) — feeding validate–retry loops.

**Q: Detect an invoice that contradicts itself?**
A: Extract `stated_total` AND `calculated_total`; flag with `conflict_detected` when they differ.

**Q: Several fixes to request — together or one at a time?**
A: Interacting fixes → one detailed batch; independent fixes → sequential.

**Q: When should the agent ask before implementing (interview pattern)?**
A: Unfamiliar domains, non-obvious implications, multiple viable approaches whose best choice depends on context.

## Domain 5 — Context & Reliability

**Q: Four error categories and the response to each?**
A: Transient → retry w/ exponential backoff; Validation → fix input, retry; Business → explain + alternative; Permission → escalate.

**Q: Why is returning an empty result on search failure an anti-pattern?**
A: Silent suppression — the coordinator can't distinguish "no matches" from "search broken".

**Q: One subagent times out mid-research. Abort?**
A: No — continue with partial results and annotate the coverage gap in the synthesis.

**Q: Three named UNRELIABLE escalation triggers?**
A: Sentiment analysis, model self-rated confidence, bolt-on automatic classifiers.

**Q: Customer says "this is outrageous!" — escalate?**
A: Not yet. Acknowledge → offer concrete resolution → escalate if they reiterate wanting a human. Explicit "get me a manager" → escalate immediately.

**Q: What does a human need at handoff?**
A: A self-contained structured summary (IDs, issue, root cause, actions taken, amounts, recommendation, reason) — they don't see the transcript.

**Q: Aggregate accuracy is 97%. Why still sample?**
A: Stratified sampling by document type/field — aggregates can hide 40% error pockets in one stratum.

**Q: Key facts keep degrading through summarization. Fix?**
A: A structured fact block (IDs, amounts, dates, status) re-included in every prompt.

**Q: Batch API: cost, window, correlation, limitation?**
A: 50% of sync cost; results within 24h — that window is the only commitment (often faster, never guaranteed), so never on a blocking path; `custom_id` per request; each request is one model invocation — no in-batch agent loop.

**Q: 100-doc batch, 5 fail on context limits. Next step?**
A: Identify failures by `custom_id`, chunk those documents, resubmit only the 5.

**Q: Result needed in 30 hours; batch takes up to 24. Submission window?**
A: 6 hours — always submit at least 24h before the deadline.

**Q: Two sources give 12% and 8%. What do you store?**
A: Both values, each with source, date, and methodology, plus `conflict_detected` — never arbitrarily pick one.

**Q: Can a batch run an agent loop that executes tools via Bash?**
A: No. Each item is one independent request; a client-side tool call must be executed by you and resubmitted as a new request. Batch single-shot analysis of pre-collected output, or run a headless `claude -p` job.

