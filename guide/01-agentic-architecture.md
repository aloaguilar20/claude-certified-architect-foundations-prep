# Domain 1 — Agentic Architecture & Orchestration (27%)

The heaviest domain on the exam. It covers how an agent actually runs: the request/response loop, the signals that control it, multi-agent topologies, deterministic guardrails, task decomposition, and session lifecycle.

**Official docs:** [Messages API](https://platform.claude.com/docs/en/api/messages) · [Handling stop reasons](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons) · [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows) · [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview) · [Subagents](https://code.claude.com/docs/en/agent-sdk/subagents) · [Hooks](https://code.claude.com/docs/en/agent-sdk/hooks) · [Sessions](https://code.claude.com/docs/en/agent-sdk/sessions)

---

## 1.1 API fundamentals

The Claude API is a stateless request–response API. A Messages API request contains:

```jsonc
{
  "model": "claude-sonnet-5",
  "max_tokens": 1024,
  "system": "You are a helpful assistant.",
  "messages": [
    {"role": "user", "content": "Hi!"},
    {"role": "assistant", "content": "Hello!"},
    {"role": "user", "content": "How are you?"}
  ],
  "tools": [],
  "tool_choice": {"type": "auto"}
}
```

- `model` — model selection (e.g. `claude-opus-5`, `claude-sonnet-5`, `claude-haiku-4-5`; check the [models overview](https://platform.claude.com/docs/en/about-claude/models/overview) for current IDs)
- `max_tokens` — response token cap
- `system` — the system prompt
- `messages` — conversation history; **you must resend the full history on every call** — the API keeps no state between requests. This single fact is the most testable thing about the request shape.
- `tools` / `tool_choice` — tool definitions and selection strategy (Domain 2)

### Message roles

Two conversational roles: `user` and `assistant`. There is **no `"tool"` role** — tool results are returned inside a *user*-role message as a `tool_result` content block referencing the `tool_use_id`:

```json
{
  "role": "user",
  "content": [
    {"type": "tool_result", "tool_use_id": "toolu_01...", "content": "..."}
  ]
}
```

See [handling tool calls](https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls).

### System prompt

Stable, request-invariant guidance belongs in `system`; per-request content belongs in the user message. Exam trap: system-prompt wording creates unintended tool associations — "always verify the customer" can cause the model to over-call `get_customer` even when unnecessary.

### stop_reason

The response's `stop_reason` says *why* generation stopped — it is the control signal for every agentic loop:

| Value | Meaning | Action |
|---|---|---|
| `end_turn` | Model finished its response | Task complete — show result |
| `tool_use` | Model wants a tool executed | Run the tool, append the result, call again |
| `max_tokens` | Token cap hit | Response truncated; raise the cap or handle |
| `stop_sequence` | A configured stop sequence fired | Application-specific |

Current docs also list `pause_turn` (long-running server-tool turn paused — send the response back to continue), `refusal`, and `model_context_window_exceeded`; see [handling stop reasons](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons). The exam focuses on the four in the table.

### Context window

Everything counts against it: system prompt + full history + tool definitions + tool results. Three named failure modes:

- **Lost-in-the-middle:** information at the start and end of long input is processed reliably; the middle is not. Mitigation: position key information near the beginning or end.
- **Tool-result accumulation:** every call appends output; a tool returning 40 fields when 5 matter wastes the window (fix: trim in a PostToolUse hook — see 1.4 and Domain 5).
- **Lossy summarization:** progressive compression turns exact numbers, dates and percentages into "about", "roughly", "a few".

---

## 1.2 The agentic loop

The core pattern of autonomous execution:

1. Send a request with tools.
2. Read `stop_reason`.
3. `tool_use` → execute the tool, append a `tool_result`, go to 1.
4. `end_turn` → done; return the result.

This is **model-driven**: Claude decides the next tool from context and prior results, unlike a hard-coded decision tree.

**Anti-patterns (named on the exam):**

- Parsing assistant text for completion phrases ("Task completed", "resolved") — the model may use those words mid-task.
- Using an arbitrary iteration cap (`max_iterations=5`) as the *primary* stop condition.
- Treating "the assistant produced text" as a completion signal.

**The only reliable completion signal is `stop_reason == "end_turn"`.**

### AgentDefinition (Agent SDK)

Subagents are configured with a definition object — name/description, a system prompt, and an explicit tool allowlist (principle of least privilege):

```python
agents = {
    "customer_support": AgentDefinition(
        description="Handles customer requests for returns and order issues",
        prompt="You are a customer support agent...",
        tools=["get_customer", "lookup_order", "process_refund", "escalate_to_human"],
    )
}
```

See the [subagents docs](https://code.claude.com/docs/en/agent-sdk/subagents) for the current field names in Python and TypeScript.

---

## 1.3 Hub-and-spoke: coordinator and subagents

```
            Coordinator
           /     |     \
   Subagent1 Subagent2 Subagent3
    (search) (analysis) (synthesis)
```

The coordinator decomposes the task, selects subagents dynamically, delegates, aggregates and validates results, handles errors/retries, and reports to the user.

**Critical principle — subagents have isolated context:**

- They do **not** inherit the coordinator's conversation history.
- All required context must be passed explicitly in the spawn prompt.
- They don't share memory across calls.
- All communication flows through the coordinator (observability, error control).

### Spawning via the Task tool

The coordinator's allowed tools must include the subagent-spawning tool (`Task`):

```text
# Bad: subagent has no context
Task: "Analyze the document"

# Good: everything it needs is in the prompt
Task: "Analyze the following document.
       Document: [full text]
       Prior search results: [results]
       Output format: [schema]"
```

**Parallel spawning:** a coordinator can request multiple Task calls in one response — those subagents run concurrently.

---

## 1.4 Hooks: deterministic guardrails

Hooks intercept the agent lifecycle with *code*, not instructions ([Agent SDK hooks](https://code.claude.com/docs/en/agent-sdk/hooks) · [Claude Code hooks](https://code.claude.com/docs/en/hooks)).

- **PreToolUse** — inspect/block/redirect a tool call before it runs (e.g. block `process_refund` above $500 and redirect to escalation).
- **PostToolUse** — transform a tool result before the model sees it (e.g. normalize dates from different MCP servers to ISO 8601; trim 40 fields to the 5 that matter).

| | Hooks | Prompt instructions |
|---|---|---|
| Guarantee | Deterministic (100%) | Probabilistic (high, never 100%) |
| Use for | Business-critical rules, financial ops, compliance | Preferences, formatting, general guidance |

**Rule:** when failure has financial, legal, or safety consequences — enforce with a hook, not a prompt.

**The converse is tested too.** Practice banks train a "hooks always win" reflex; the exam punishes it:

- *Mandatory sequence skipped* (verify identity before refund) → programmatic enforcement.
- *Judgment miscalibrated* (wrong escalations, unactionable review comments, too many false positives) → explicit criteria and few-shot examples. A hook can't encode a judgment boundary, and forcing one usually breaks a legitimate case.

**Proportionality principle:** pick the least complex control that addresses the identified failure. Distractors are frequently over-engineered — routing layers, trained classifiers, second model instances — when a better tool description or clearer criterion fixes the real cause.

---

## 1.5 Task decomposition

**Fixed pipeline (prompt chaining):** steps known up front, predictable structure, reproducibility matters. `Document → metadata → extraction → validation → enrichment → output`.

**Dynamic adaptive decomposition:** subtasks emerge from intermediate results — open-ended investigations, unknown scope, step N depends on step N-1's findings. ("Add tests to a legacy codebase" → map structure → prioritize risky module → discover an external API dependency → adapt by mocking it first.)

**Multi-pass code review:** for a PR touching 10+ files, review each file individually (local issues), then run an integration pass (cross-file issues). A single pass over 14 files fails by **attention dilution**: deep analysis on some files and shallow on others, inconsistent comments (a pattern flagged in one file, approved in another), missed obvious bugs.

---

## 1.6 Sessions: continue, resume, fork

([Sessions docs](https://code.claude.com/docs/en/agent-sdk/sessions))

- **Continue** — pick up the most recent session in the directory, no ID tracking.
- **Resume** (`resume=<session-id>`) — return to a *specific* session with its full context. Risk: if files changed since, tool results in that history are stale.
- **Fork** (`resume=<id>` + `fork_session=True` in Python / `forkSession: true` in TypeScript) — new session that copies history up to the branch point, then diverges; the original is untouched. Use to compare approaches (e.g. Redux vs Context API) from shared investigation context.

**Start a fresh session instead when:** tool results are stale, or context has degraded — "here is a short summary of what we found" beats resuming old tool data.

---

## Exam reflexes for this domain

- Loop control = `stop_reason`, never text matching or iteration caps.
- Subagents inherit **nothing** — context is passed explicitly, and the coordinator needs `Task` in its allowed tools.
- Financial/legal/safety consequence → hook. Judgment miscalibration → criteria/examples. Always the least complex fix.
- Many files, one pass → attention dilution → decompose.
