# Domain 3 — Claude Code Configuration & Workflows (20%)

Configuring Claude Code for a team: persistent instructions, scoped rules, skills, plan mode, the built-in tools, and headless CI/CD usage.

**Official docs:** [Memory & CLAUDE.md](https://code.claude.com/docs/en/memory) · [Skills](https://code.claude.com/docs/en/skills) · [Subagents](https://code.claude.com/docs/en/sub-agents) · [Headless mode](https://code.claude.com/docs/en/headless) · [GitHub Actions](https://code.claude.com/docs/en/github-actions)

---

## 3.1 The CLAUDE.md hierarchy

CLAUDE.md files give Claude persistent instructions, loaded at session start. Locations, broadest to most specific ([memory docs](https://code.claude.com/docs/en/memory)):

| Scope | Location | Shared via VCS? | Use for |
|---|---|---|---|
| Managed policy | e.g. `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) | Deployed by IT | Org-wide standards |
| **User** | `~/.claude/CLAUDE.md` | **No** | Personal preferences, all projects |
| **Project** | `./CLAUDE.md` or `./.claude/CLAUDE.md` | **Yes** | Team standards, architecture, workflows |
| Local | `./CLAUDE.local.md` (gitignored) | No | Personal project-specific prefs |
| Directory | `CLAUDE.md` in subdirectories | Yes | Loaded on demand when Claude works with files there |

**The classic exam trap:** a new teammate doesn't get the team's conventions because someone put them in *user-level* `~/.claude/CLAUDE.md` instead of a *project-level* CLAUDE.md in version control.

### @path imports

```markdown
Coding standards: @./standards/coding-style.md
Project overview: @README.md, dependencies: @package.json
```

- `@` immediately before the path (no space); relative and absolute paths supported.
- Relative paths resolve **relative to the file containing the import**, not the working directory.
- Recursive imports up to a **maximum depth of four hops** (current docs; older study materials say five — the docs changed).

## 3.2 .claude/rules/ — scoped rules

An alternative to a monolithic CLAUDE.md: one topic per file (`testing.md`, `api-conventions.md`…). Rules **without** frontmatter load every session; rules **with** `paths:` frontmatter load only when Claude works with matching files — saving context:

```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
---
Tests use describe/it blocks. Use data factories; don't mock the database.
```

**Rules vs directory CLAUDE.md:** use `paths`-scoped rules when a convention applies to files *scattered across* the tree (tests, migrations); use a directory CLAUDE.md when conventions belong to one directory.

## 3.3 Slash commands and skills

**Custom commands have been merged into skills** ([skills docs](https://code.claude.com/docs/en/skills)): `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy`, and the legacy format keeps working. Project-level skills live in VCS and give the whole team the same workflows; personal ones live in `~/.claude/skills/`.

Skills add frontmatter configuration. The exam-relevant fields:

| Field | Effect |
|---|---|
| `context: fork` | Runs the skill in an isolated subagent — verbose output doesn't pollute the main session |
| `allowed-tools` | Pre-approves the listed tools for the skill's turn (least privilege) |
| `argument-hint` | Autocomplete hint for expected arguments, e.g. `[path]` |
| `disable-model-invocation: true` | Only you can trigger it via `/name`; Claude won't auto-invoke |

**Skill vs CLAUDE.md:** a skill is an on-demand procedure, loaded only when used; CLAUDE.md is always-loaded facts and standards. If a CLAUDE.md section has grown into a multi-step procedure, move it to a skill.

## 3.4 Plan mode vs direct execution

**Plan mode:** Claude only investigates (Read/Grep/Glob) and produces a plan for approval — no changes, no side effects. Use for: large changes (dozens of files), multiple plausible approaches, architectural decisions, unfamiliar codebases, wide migrations.

**Direct execution:** single-file fixes with a clear stack trace, one validation check, well-understood unambiguous changes.

**Combined:** plan → user approves → execute the approved plan.

The **Explore subagent** isolates verbose codebase exploration and returns only a summary — protecting the main context window in multi-phase work ([subagents](https://code.claude.com/docs/en/sub-agents)).

## 3.5 /compact and /memory

- **/compact** summarizes prior history to free the context window in long sessions. Risk: exact numbers, dates and specifics can be lost in summarization. (Project-root CLAUDE.md is re-injected after compaction; nested/path-scoped rules reload on next matching file access.)
- **/memory** opens the memory files (CLAUDE.md and friends) for editing — persist notes, preferences, and conventions across sessions instead of re-explaining every time.

## 3.6 Built-in tools

| Task | Tool |
|---|---|
| Find files by name/pattern | **Glob** (`**/*.test.tsx`) |
| Search file contents | **Grep** (function name, error message, import) |
| Read a file | **Read** |
| Create a file | **Write** |
| Precise in-place change | **Edit** (unique text match) |
| Shell commands (git, npm, tests) | **Bash** |

**Incremental investigation:** don't read everything at once. Grep entry points → Read those files → Grep usages → Read consumers → repeat.

**Edit fallback:** if Edit fails on a non-unique match → Read the file, modify programmatically, Write it back.

## 3.7 Claude Code in CI/CD

Headless mode ([docs](https://code.claude.com/docs/en/headless)):

```bash
claude -p "Review this PR for security issues" \
  --output-format json --json-schema '{"type":"object", ...}'
```

- `-p` / `--print` — non-interactive: process the prompt, print, exit. The correct way to run Claude in a pipeline.
- `--output-format json` + `--json-schema` — machine-parseable, schema-validated output for posting inline PR comments.

**Session-context isolation:** the session that *generated* code is worse at *reviewing* it — it retains its own reasoning and won't challenge its own decisions. Use an independent instance for review.

**Duplicate comments on re-review:** include the prior review results in context and instruct Claude to report only new or unresolved issues.

---

## Exam reflexes for this domain

- "Teammate didn't get the instructions" → they were user-level; move to project-level in VCS.
- Scattered file types → `paths`-scoped rule; one directory → directory CLAUDE.md; procedure → skill; always-true fact → CLAUDE.md.
- Big/ambiguous/unfamiliar → plan mode. Small/clear → direct.
- CI → `-p` with JSON output + schema; separate instance for review.
