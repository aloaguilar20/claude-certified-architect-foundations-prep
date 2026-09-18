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

## 3.6 Built-in tools and codebase exploration

| Task | Tool |
|---|---|
| Find files by name/pattern | **Glob** (`**/*.test.tsx`) — matches paths only |
| Search file contents | **Grep** (function name, error message, import) |
| Read a file (or a slice of it) | **Read** (`offset` / `limit`) |
| Create a file | **Write** |
| Precise in-place change | **Edit** (unique text match) |
| Shell commands (git, npm, tests) | **Bash** |

([Tools reference](https://code.claude.com/docs/en/tools-reference))

**The reflex:** Glob answers *"which files?"* by **name**; Grep answers *"where is this?"* by **content**. Finding the callers of `calculateTax` is a Grep job — a Glob for `*tax*` misses every caller whose filename doesn't say "tax".

### The token-efficient exploration loop

1. **Locate cheaply.** Grep with file-list output (the default) or `count` first; scope with `path`, `glob` or `type`; cap noisy output with `head_limit`. Switch to `content` mode only once the search is narrow.
2. **Read narrowly.** Read just the files that matter — for a huge file, Grep with line numbers, then Read a window with `offset`/`limit`.
3. **Follow the thread.** Grep usages of what you learned → Read consumers → repeat. Build understanding incrementally instead of reading everything up front.
4. **Trace through re-exports.** If a Grep for a name only hits a barrel/`index` file, read the barrel, collect the exported names (they may be aliased), then Grep each one.
5. **Use dedicated tools, not shell pipelines.** `grep -r | head` and `cat` through Bash produce unbounded, unstructured output. Reserve Bash for what needs a shell: git history, running tests, builds.

### Keeping a long exploration alive

- **Delegate verbose searching** to the read-only **Explore** subagent: it works in its own context and returns a summary, so the main context stays clean.
- **Write findings to a scratchpad file** (structure, entry points, open questions). It survives `/compact`, context limits and session boundaries.
- **On resume,** start from the scratchpad plus a targeted re-check of files that changed since (`git diff`), rather than re-exploring from scratch.

| Anti-pattern | Better |
|---|---|
| Read every file to find usages | Grep → Read the hits |
| Glob to find callers | Grep (contents), Glob only for name patterns |
| Grep in `content` mode across the whole repo first | Files-only/count → narrow → content |
| `cat`/`grep -r` via Bash | Read / Grep |
| Read a 6,000-line file whole | Grep `-n`, then Read `offset`/`limit` |

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

### Configuring an automated review (the pattern to memorize)

A CI review job has three parts, and exam questions test all three:

1. **Load the right standards** — review criteria live in the **project-level** `CLAUDE.md` checked into the repo, so every run and every teammate gets them (not in a user-level file on someone's laptop).
2. **Restrict tool access** — a reviewer reads; it must not edit. Use a read-only allowlist (`--allowedTools "Read,Grep,Glob"`, or `--tools` to limit what exists at all).
3. **Structured output** — `--output-format json` with `--json-schema`; the validated result comes back in the `structured_output` field, ready to turn into inline PR comments.

```bash
claude -p "Review this PR against our standards" \
  --allowedTools "Read,Grep,Glob" \
  --output-format json --json-schema "$(cat review-schema.json)" \
  --max-turns 15 --max-budget-usd 2
```

**Runaway protection for unattended runs:** `--max-turns` and `--max-budget-usd` (print mode) bound the loop; a minimal allowlist bounds the blast radius. `--dangerously-skip-permissions` removes a safeguard — it is never the answer to "the job keeps stopping."

### Testing strategies

- **Give context, not adjectives.** For test generation, supply existing test files, name the fixture conventions to reuse, and define what makes a test meaningful (asserts behavior, covers edge cases, fails if the logic breaks). "More thorough" produces more of the same.
- **Tests are the feedback loop.** Run the suite and hand Claude the *failing output* (expected vs actual). Concrete failures beat "try again."
- **Bugs: reproduce first.** Write a failing test that reproduces the report → fix until it and the suite pass. The test verifies the fix and stays as a regression guard.
- **Communicating issues:** interacting problems go in **one** message (one coherent fix); independent problems go as separate, focused requests.
- **Input/output examples** beat prose for transformations — when a description keeps being misread, show 2–3 concrete before/after pairs.

### Finding more bugs in review

- **Independent reviewer:** a fresh session or subagent reviews the diff — the authoring session shares its own blind spots.
- **Multi-pass for big PRs:** per-file passes for local issues, then a separate cross-file integration pass. One pass over 14 files loses depth.
- **Cut false positives with explicit criteria:** state what to flag (bugs, security), what to skip (style nits, accepted patterns), and feed prior findings back in so re-reviews report only new or unresolved issues.

### PR merging: deterministic gates vs. probabilistic review

An AI reviewer that says "LGTM" is **advisory**. What actually gates a merge is deterministic — required CI checks (tests, lint, build) — plus human approval for risky areas. Hooks (e.g. a `PostToolUse` formatter/linter hook) enforce rules on every edit; CLAUDE.md only *guides*. Never let the session that wrote the code approve its own PR.

### Batch or synchronous in a pipeline?

Blocking pre-merge check → **synchronous**. Nightly or weekly bulk report → **Batches API** (50% cheaper, results within 24h — the only commitment). And no agent loops in a batch: each item is one independent request, so a client-side tool call can't be executed and continued inside it.

---

## Exam reflexes for this domain

- "Teammate didn't get the instructions" → they were user-level; move to project-level in VCS.
- Scattered file types → `paths`-scoped rule; one directory → directory CLAUDE.md; procedure → skill; always-true fact → CLAUDE.md.
- Big/ambiguous/unfamiliar → plan mode. Small/clear → direct.
- CI → `-p` with JSON output + schema; separate instance for review.
- Review config = project CLAUDE.md standards + read-only tools + JSON schema output. Runaway control = `--max-turns` / `--max-budget-usd`.
- Glob = names, Grep = contents, Read a slice; narrow before you read. Bash is for git/tests/builds.
- Failing tests → give Claude the failure output. Bug → failing test first. AI "LGTM" never replaces required checks + human approval.
