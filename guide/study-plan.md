# 28-Day Study Plan

Four weeks from zero to exam-ready. Reading days reference the [domain chapters](./); practice references the [interactive exam](../practice-exam.html). The [interactive study reader](../study-guide.html) mirrors this plan day by day with progress tracking.

## Week 1 — Domain 1: Agentic Architecture & Orchestration (27%)

| Day | Topic | Chapter section |
|---|---|---|
| 1 | API fundamentals: request shape, roles, `stop_reason`, system prompt, context window | [1.1](01-agentic-architecture.md#11-api-fundamentals) |
| 2 | The agentic loop & AgentDefinition — *the* Domain 1 concept | [1.2](01-agentic-architecture.md#12-the-agentic-loop) |
| 3 | Hub-and-spoke, subagent context isolation, the Task tool, parallel spawning | [1.3](01-agentic-architecture.md#13-hub-and-spoke-coordinator-and-subagents) |
| 4 | Hooks: deterministic vs probabilistic, and the converse trap | [1.4](01-agentic-architecture.md#14-hooks-deterministic-guardrails) |
| 5 | Task decomposition: pipelines vs dynamic; multi-pass review | [1.5](01-agentic-architecture.md#15-task-decomposition) |
| 6 | Sessions: continue / resume / fork; when to start fresh | [1.6](01-agentic-architecture.md#16-sessions-continue-resume-fork) |
| 7 | **Checkpoint** — Domain 1 questions in the practice exam; re-read anything missed | — |

## Week 2 — Domains 2 & 3: Tools/MCP (18%) + Claude Code (20%)

| Day | Topic | Chapter section |
|---|---|---|
| 8 | Tool definitions, descriptions as the interface, `tool_choice` | [2.2–2.3](02-tool-design-mcp.md) |
| 9 | MCP primitives, servers, project vs user configuration | [2.4](02-tool-design-mcp.md#24-mcp-the-open-protocol) |
| 10 | MCP error handling: structured `isError` payloads | [2.5](02-tool-design-mcp.md#25-mcp-error-handling--the-iserror-flag) |
| 11 | CLAUDE.md hierarchy & @path imports | [3.1](03-claude-code.md#31-the-claudemd-hierarchy) |
| 12 | `.claude/rules/`, slash commands & skills | [3.2–3.3](03-claude-code.md#32-clauderules--scoped-rules) |
| 13 | Plan mode, /compact, /memory, built-in tools | [3.4–3.6](03-claude-code.md#34-plan-mode-vs-direct-execution) |
| 14 | Claude Code in CI/CD: headless mode, JSON output, review isolation | [3.7](03-claude-code.md#37-claude-code-in-cicd) |

## Week 3 — Domains 4 & 5: Prompt Engineering (20%) + Context/Reliability (15%)

| Day | Topic | Chapter section |
|---|---|---|
| 15 | JSON-schema structured output; syntax vs semantic errors | [4.1](04-prompt-engineering.md#41-structured-output-with-json-schemas) |
| 16 | Few-shot, explicit criteria, chaining, interview pattern | [4.2–4.4](04-prompt-engineering.md#42-few-shot-prompting) |
| 17 | Validation + retry-with-feedback; Pydantic; self-correction | [4.5–4.6](04-prompt-engineering.md#45-validation-and-retry-with-feedback) |
| 18 | Message Batches API + SLA math | [5.5](05-context-reliability.md#55-message-batches-api) |
| 19 | Escalation & human-in-the-loop | [5.2](05-context-reliability.md#52-escalation-and-human-in-the-loop) |
| 20 | Error handling + context management strategies | [5.1, 5.3](05-context-reliability.md) |
| 21 | Preserving provenance | [5.4](05-context-reliability.md#54-preserving-provenance) |

## Week 4 — Practice & remediation (no new reading)

A diagnose → repair → retest loop:

| Day | Task | Done when |
|---|---|---|
| 22 | **Baseline simulation** — timed, 4-of-6 scenarios, no feedback until the end. No warm-up: an honest baseline is the point. | A scaled score you trust + the app's ranked review plan |
| 23 | **Work the review plan** — re-read *only* the sections you missed | Every flagged section re-read |
| 24 | **Weakest two domains, weighted by exam share** (a D1 gap costs ~2× a D5 gap) | Cause-level understanding, not memorized answers |
| 25 | **Second simulation**, different scenario draw; compare per-domain with Day 22 | Confirmation the same misconception isn't recurring |
| 26 | **Build one artifact per weak domain** — a loop trace with `stop_reason` handling, a tool contract with an error matrix, a repo with a project skill + path-scoped rule | One runnable/written artifact per weak domain |
| 27 | **Drills & consolidation** — 10-question quick drills, [cheat sheet](../cheatsheets/cheat-sheet.md), [flashcards](../cheatsheets/flashcards.md) | Recall speed, calm head |
| 28 | **Final run + logistics** — one last timed simulation, nothing blank; verify registration, retake rules, test-center/proctoring setup. Stop early. Rest. | Ready |

See [00-exam-guide.md](00-exam-guide.md) for the answering method and readiness checklist.
