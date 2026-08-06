# Claude Certified Architect — Foundations (CCAR-F) Study Repo

Open study materials for Anthropic's **Claude Certified Architect – Foundations** certification: a domain-by-domain written guide, a 28-day study plan, cheat sheets, flashcards, and a **free interactive practice exam** with scaled scoring and a personalized review plan.

> **Not affiliated with or endorsed by Anthropic.** This is independent, community-built study material. Exam names are Anthropic trademarks. Always verify logistics against the official exam guide and certification pages before scheduling.

## 👋 About me — why this exists

I'm **Alonso Aguilar**, a project management professional with **14 years in the IT world**, now doing what I'm most passionate about: **transitioning into AI as a professional** — not by reading about it, but by building with it every day.

I work on independent projects as a **founder**, with **Claude Code** at the center of my workflow — projects like [**Biblicuentos**](https://biblicuentos.com) and [**SOHpro**](https://sohpro.app), an EV battery health platform. Preparing for the CCAR-F exam, I ended up building the study material I wished existed: organized by exam domain, checked against the current official docs, with a practice exam that simulates real scoring. Publishing it felt more useful than keeping it in a folder.

I keep learning and adapting as a professional every single day — and honestly, that's the part I enjoy most. If this material helps you, I'd love to hear about it: **[connect with me on LinkedIn](https://www.linkedin.com/in/alonso-aguilar-araya/)**. And if you spot something outdated, [contribute](#-contributing) — this stays useful only if the community keeps it current.

**Pura vida** 🇨🇷

## 🚀 Start here

| Resource | What it is |
|---|---|
| **[Interactive practice exam](https://aloaguilar20.github.io/claude-certified-architect-foundations-prep/practice-exam.html)** | 51 scenario-anchored questions · study / timed-simulation / quick-drill modes · scaled 100–1,000 scoring at the 720 line · generates a ranked review plan |
| **[Interactive study reader](https://aloaguilar20.github.io/claude-certified-architect-foundations-prep/study-guide.html)** | The full guide as a day-by-day reader with progress tracking, deep-linked from every exam question |
| [Exam guide & strategy](guide/00-exam-guide.md) | Format, scoring, retakes, the six scenarios, and a 5-step method for scenario questions |
| [Domain 1 — Agentic Architecture & Orchestration (27%)](guide/01-agentic-architecture.md) | The agentic loop, subagents, hooks, decomposition, sessions |
| [Domain 2 — Tool Design & MCP Integration (18%)](guide/02-tool-design-mcp.md) | Tool descriptions, `tool_choice`, MCP servers, structured errors |
| [Domain 3 — Claude Code Configuration & Workflows (20%)](guide/03-claude-code.md) | CLAUDE.md, rules, skills, plan mode, built-in tools, CI/CD |
| [Domain 4 — Prompt Engineering & Structured Output (20%)](guide/04-prompt-engineering.md) | JSON schemas, few-shot, explicit criteria, validation & retry |
| [Domain 5 — Context Management & Reliability (15%)](guide/05-context-reliability.md) | Error handling, escalation, context strategies, provenance, batches |
| [28-day study plan](guide/study-plan.md) | A four-week schedule through all of the above |
| [Cheat sheet](cheatsheets/cheat-sheet.md) · [Flashcards](cheatsheets/flashcards.md) | Last-week consolidation material |

## 📋 Exam at a glance

| Item | Detail |
|---|---|
| Exam code | **CCAR-F** on Pearson VUE (formerly CCA-F on the original platform — same content) |
| Format | 60 items, 120 minutes; 4 scenarios drawn at random from a published set of 6 |
| Question types | Multiple choice and multiple response |
| Scoring | Scaled 100–1,000; **720 to pass** (scaled — 720 ≠ 72% correct) |
| Retakes | 14 days after a 1st failure, 30 after a 2nd, 90 after a 3rd; four attempts per rolling 12 months |
| Validity | 12 months |
| Access | Via the Claude Partner Network / Partner Academy, scheduled through Pearson VUE |

**Domain weightings:** D1 Agentic Architecture 27% · D2 Tools/MCP 18% · D3 Claude Code 20% · D4 Prompt Engineering/Structured Output 20% · D5 Context/Reliability 15%.

*Logistics change; verify on the official certification page before you book.*

## 📚 Sources & attribution

- Written against **Anthropic's official CCAR-F exam guide** and the **official documentation**, which is cited per topic throughout:
  - [Claude API & platform docs](https://platform.claude.com/docs) (Messages API, tool use, batches, prompt engineering)
  - [Claude Code & Agent SDK docs](https://code.claude.com/docs) (CLAUDE.md, skills, hooks, subagents, sessions, headless)
  - [Model Context Protocol](https://modelcontextprotocol.io) (tools, resources, prompts, servers)
- Topic sequencing was originally inspired by the excellent community guide [paullarionov/claude-certified-architect](https://github.com/paullarionov/claude-certified-architect) — a great complementary resource, with guides in 11 languages. The text here was written independently, reorganized by exam domain, and updated against the current official docs (see the "current-docs notes" flagged throughout, e.g. `pause_turn`, the four-hop import limit, and batch-API tool support).

## 🖥 Run the interactive material locally

Both HTML apps are single files with no build step and no network calls — clone the repo and open `practice-exam.html` or `study-guide.html` in a browser. Progress is stored locally in your browser.

## 🤝 Contributing

Spotted something outdated or disputed? Open an issue with a link to the official doc page that contradicts the text. Corrections with primary-source citations are merged fast.

## 📄 License

[MIT](LICENSE) — reuse freely with attribution.

---

Maintained by **Alonso Aguilar** · [LinkedIn](https://www.linkedin.com/in/alonso-aguilar-araya/) — connect and tell me how the exam went! If this helped you pass, a ⭐ helps others find it.
