# Exam Guide & Strategy — CCAR-F

> Independent study material; not affiliated with or endorsed by Anthropic. Verify all logistics on the official certification page before scheduling.

## Logistics

| Item | Current information |
|---|---|
| Exam code | **CCAR-F** on Pearson VUE. The older **CCA-F** code is from the original delivery platform and still appears across third-party material — same content. |
| Format | 60 items, 120 minutes (~135 min seat time). Four scenarios drawn at random from a published set of six. |
| Question types | Multiple choice **and multiple response**. Older materials describe only one-correct-of-four; practice both. |
| Scoring | Scaled 100–1,000, minimum **720**. Scaled means 720 is *not* 72% correct; no raw-score conversion or pass rate is published. Treat any site claiming an exact required question count as invented. |
| Retakes | 14 days after a first failure, 30 after a second, 90 after a third. Four attempts per rolling 12 months. |
| Validity | 12 months from the award date. |
| Access | Open to organizations in the [Claude Partner Network](https://claude.com/partners) — membership is free. Register through the Partner Academy, then schedule via Pearson VUE. |
| Official exam guide | **Get it and read it first.** Not publicly hosted: after your org joins the Partner Network, log into the Anthropic Partner Academy and download it from the certification page. It is the authoritative source for logistics, domains, and the six scenarios — everything in this repo defers to it. |

## The five domains

| # | Domain | Weight | Chapter |
|---|---|---|---|
| D1 | Agentic Architecture & Orchestration | **27%** | [Chapter 1](01-agentic-architecture.md) |
| D2 | Tool Design & MCP Integration | 18% | [Chapter 2](02-tool-design-mcp.md) |
| D3 | Claude Code Configuration & Workflows | 20% | [Chapter 3](03-claude-code.md) |
| D4 | Prompt Engineering & Structured Output | 20% | [Chapter 4](04-prompt-engineering.md) |
| D5 | Context Management & Reliability | 15% | [Chapter 5](05-context-reliability.md) |

A gap in D1 costs roughly twice what the same gap costs in D5 — weight your review accordingly.

## The six published scenarios

Every exam item is anchored to one of six scenario contexts; your sitting draws four:

1. **Customer support resolution agent** (Agent SDK + custom MCP tools; escalation targets)
2. **Code generation with Claude Code** (slash commands, CLAUDE.md, plan mode vs direct execution)
3. **Multi-agent research system** (coordinator + search/analysis/synthesis subagents; cited reports)
4. **Developer productivity agent** (built-in tools, unfamiliar codebases, MCP integration)
5. **Claude Code in CI/CD** (automated review, actionable feedback, minimizing false positives)
6. **Structured data extraction** (JSON schema validation, edge cases, downstream integration)

Learn to map each scenario to its dominant domains — questions frequently test D2 vs D3 vs D4 boundaries.

## A 5-step method for scenario questions

1. **Name the required outcome.** Deterministic compliance? Better tool selection? Fewer false positives? Recoverability?
2. **Find the binding constraint.** Words like *must*, *first*, *shared with the team*, *without interaction*, and *when policy is silent* usually decide the answer.
3. **Name the failure mode.** A wrong tool, stale context, ambiguous policy, invalid schema output, and a transient outage each demand a *different* fix.
4. **Choose the least complex option that solves that failure.** Better descriptions often beat a new router; explicit criteria often beat a new classifier. If prompt optimization hasn't been tried, an ML pipeline is almost never the keyed answer.
5. **Test every distractor.** State what it changes and why that doesn't satisfy the constraint. On multiple-response items, judge each option independently.

## Timing and guessing

- Two minutes per question on average; at the 60-minute mark you should be near question 30.
- Unanswered items are scored wrong and there is **no guessing penalty** — never leave a blank. Flag, move on, reserve a final review window.

## Readiness check

You are ready when you can:

- map each of the six scenarios to its primary domains without confusing D2, D3 and D4;
- explain **why each distractor fails**, not merely identify the keyed answer;
- complete a fresh mixed set without repeating the same failure pattern;
- finish 60 questions in 120 minutes with nothing left blank;
- produce one practical artifact per domain (an agent-loop trace, a tool contract with an error matrix, a repo with a project skill and a path-scoped rule…).

Aim for a consistent 80%+ in every domain before booking — the margin absorbs scaling uncertainty and the fact that practice questions are never the real ones.

## Official primary sources

- [Claude platform docs](https://platform.claude.com/docs) — Messages API, tool use, structured output, batches, prompt engineering
- [Claude Code docs](https://code.claude.com/docs) — CLAUDE.md/memory, skills, hooks, subagents, headless/CI, Agent SDK
- [Model Context Protocol](https://modelcontextprotocol.io) — protocol spec, server concepts
- The official CCAR-F exam guide (distributed through the Partner Academy)

*(The older docs.anthropic.com paths redirect to these domains, so old bookmarks keep working.)*
