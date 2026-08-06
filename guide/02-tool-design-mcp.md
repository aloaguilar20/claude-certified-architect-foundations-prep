# Domain 2 — Tool Design & MCP Integration (18%)

How Claude selects and calls external functions, and how the Model Context Protocol standardizes connecting external systems.

**Official docs:** [Tool use overview](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview) · [Implementing tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use) · [MCP intro](https://modelcontextprotocol.io) · [MCP architecture](https://modelcontextprotocol.io/docs/learn/architecture) · [MCP in Claude Code](https://code.claude.com/docs/en/mcp)

---

## 2.1 tool_use mechanics

Claude never executes code itself. It emits a structured `tool_use` request; *your* code runs the function and returns a `tool_result`. (Loop mechanics: [Domain 1](01-agentic-architecture.md).)

## 2.2 Tool definitions — the description is the interface

```json
{
  "name": "get_customer",
  "description": "Finds a customer by email or ID. Returns the customer profile including name, email, order history, and account status. Use this tool BEFORE lookup_order to verify the customer's identity. Accepts an email (format: user@domain.com) or a numeric customer_id.",
  "input_schema": {
    "type": "object",
    "properties": {
      "email": {"type": "string", "description": "Customer email"},
      "customer_id": {"type": "integer", "description": "Numeric customer ID"}
    },
    "required": []
  }
}
```

**The description is the primary selection mechanism.** The model chooses tools by reading descriptions; minimal ones ("Retrieves customer information") cause mistakes as soon as tools overlap. A good description covers:

- What the tool does and returns
- Input formats with example values
- Edge cases and constraints
- **When to use it vs similar alternatives**

Avoid near-identical descriptions across tools (`analyze_content` vs `analyze_document`) — the model will confuse them.

**Built-in vs MCP preference:** agents may prefer built-in tools (Read, Grep) over an MCP tool with similar function. The keyed fix is to *strengthen the MCP tool's description* — state its unique data and advantages — not to remove the built-in tool or add a routing layer.

**One tool doing five jobs** (summarize + extract + classify + …) with behavior chosen by guesswork → split into purpose-specific tools with clear contracts. Description quality and tool granularity beat routers and classifiers.

## 2.3 tool_choice

| Value | Behavior | Use when |
|---|---|---|
| `{"type": "auto"}` | Model decides: tool call or text | Default |
| `{"type": "any"}` | Model **must** call *some* tool | Guaranteed structured output, model picks the best schema/tool |
| `{"type": "tool", "name": "..."}` | Model **must** call *that* tool | Forced first step / fixed execution order |
| `{"type": "none"}` | No tool calls — text only | Tools defined in context but this turn must answer in prose |

Classic scenarios: `any` + three extraction schemas of unknown document type → model selects the right schema but output is guaranteed structured; forced selection → `extract_metadata` always runs before enrichment.

## 2.4 MCP: the open protocol

MCP standardizes connecting external systems to Claude. Three primitives ([architecture](https://modelcontextprotocol.io/docs/learn/architecture)):

- **Tools** — functions the agent calls to *act* (CRUD, API calls, execution)
- **Resources** — data the agent *reads* for context (docs, schemas, catalogs)
- **Prompts** — predefined prompt templates for common tasks

**Resource advantage:** the agent gets an immediate "map" (e.g. a database schema or task catalog) without spending exploratory tool calls.

### MCP servers

A server is a process implementing the protocol. On connect, its tools are discovered automatically; tools from all connected servers are available at once; their descriptions determine usage.

### Configuration in Claude Code

| Scope | File | Traits |
|---|---|---|
| Project (team) | `.mcp.json` at repo root | Checked into version control; secrets via env-var expansion (`${GITHUB_TOKEN}`) — tokens are never committed; applies to all contributors |
| User (personal) | user-scope config (`~/.claude.json`) | Not shared; personal/experimental servers |

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {"GITHUB_TOKEN": "${GITHUB_TOKEN}"}
    }
  }
}
```

See [MCP in Claude Code](https://code.claude.com/docs/en/mcp) for current scopes and syntax.

**Choosing servers:** for standard integrations (GitHub, Jira, Slack) prefer existing community servers; build custom servers only for unique, team-specific workflows.

## 2.5 MCP error handling — the isError flag

A failing MCP tool sets `isError: true`. What matters is the *content* of the error:

**Structured (good):**

```json
{
  "isError": true,
  "content": {
    "errorCategory": "transient",
    "isRetryable": true,
    "message": "Timeout while calling the orders API.",
    "attempted_query": "order_id=12345",
    "partial_results": null
  }
}
```

**Generic (anti-pattern):** `{"isError": true, "content": "Operation failed"}` — the agent can't decide whether to retry, rephrase, or escalate.

Include: error category, retryability, a human-readable message, the attempted input, and any partial results. (This pattern generalizes to subagent errors — see [Domain 5](05-context-reliability.md).)

---

## Exam reflexes for this domain

- Tool-selection problems → fix the **description** first; routers/classifiers are usually distractor over-engineering.
- Need guaranteed structure but flexible choice → `any`. Need a fixed first action → forced `tool`.
- Team-shared MCP config → `.mcp.json` in VCS with env-var secrets; personal experiments → user scope.
- Errors must carry enough structure for the *agent* to choose a recovery path.
