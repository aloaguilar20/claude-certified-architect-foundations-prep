# Domain 4 — Prompt Engineering & Structured Output (20%)

Getting reliably correct, reliably *shaped* output: JSON schemas via tool use, few-shot examples, explicit criteria, chaining, clarifying questions, and validation loops.

**Official docs:** [Prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) · [Tool use for structured output](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview) · [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook)

---

## 4.1 Structured output with JSON schemas

Using `tool_use` with a JSON schema is the most reliable way to get structured output. It **guarantees syntax** (valid JSON, required fields present) but **not semantics** (values can still be wrong).

Schema design rules that get tested:

1. **`required` only when the data is always present.** Marking a sometimes-absent field required pushes the model to *fabricate* values.
2. **Nullable for maybe-absent data:** `"type": ["string", "null"]` lets the model honestly return null instead of hallucinating.
3. **Enums with `"other"` + a detail field** so out-of-category data isn't forced into a wrong bucket.
4. **An `"unclear"` enum value** where the model may lack confidence — an honest "unclear" beats a confidently wrong category.

```json
{
  "type": "object",
  "properties": {
    "category": {"type": "string", "enum": ["bug", "feature", "docs", "unclear", "other"]},
    "category_detail": {"type": ["string", "null"], "description": "Details if category is 'other' or 'unclear'"},
    "severity": {"type": "string", "enum": ["critical", "high", "medium", "low"]},
    "confidence": {"type": "number", "minimum": 0, "maximum": 1}
  },
  "required": ["category", "severity"]
}
```

### Syntax vs semantic errors

| Type | Example | Mitigation |
|---|---|---|
| Syntax | Invalid JSON, wrong type, missing brace | Schema-backed tool use (eliminates the class) |
| Semantic | Totals don't add up, value in wrong field, hallucinated value | Validation checks, retry-with-feedback, self-correction |

## 4.2 Few-shot prompting

2–4 input/output examples demonstrating expected behavior. Why examples beat prose: "be more precise" is interpretable many ways; an example shows the format *and the decision logic*, and the model generalizes the pattern.

Use examples for:

- **Ambiguous scenarios:** "My order is broken" → look up the order (could mean damaged item); "Get me a manager" → escalate immediately, don't attempt to solve.
- **Output formatting:** one complete example finding with location/issue/severity/fix.
- **Boundary drawing:** show a snippet to flag *and* a similar one not to flag (`== true` vs a clean `.filter(x => x.active)`).
- **Format variety in extraction:** inline citations vs bibliography references; informal measurements ("two handfuls" → `~100g, precision: approximate`) — too diverse for rules, ideal for examples.

**Normalization rules** belong in the prompt alongside strict schemas: dates → ISO 8601 ("yesterday" → absolute date); currency → amount + code ("five bucks" → `{"amount": 5, "currency": "USD"}`); percentages → decimals. Prevents semantically inconsistent but syntactically valid output.

## 4.3 Explicit criteria vs vague instructions

Vague: "Check comments for accuracy. Be conservative." Explicit:

```text
Flag a comment ONLY if:
1. It describes behavior that CONTRADICTS the actual code
2. It references a non-existent function or variable
3. A TODO/FIXME refers to a bug already fixed

Do NOT flag: stylistically outdated comments, minor wording issues, missing comments.
```

Define severities with examples: CRITICAL = user-facing runtime failure (NPE in payment path); HIGH = security vulnerability (SQLi, XSS, missing authz); MEDIUM = logic bug without immediate impact (off-by-one); LOW = code quality (duplication).

**When the failure is miscalibrated judgment — too many false positives, wrong escalations — the fix is criteria and examples, not hooks or classifiers.**

## 4.4 Prompt chaining and iterative workflows

**Chaining:** split a complex task into focused sequential steps (per-file review passes → integration pass) to avoid attention dilution ([Domain 1](01-agentic-architecture.md#15-task-decomposition)). Chaining suits predictable, repeatable tasks; dynamic decomposition suits open-ended investigation.

The exam also names these iteration patterns (grouped under Claude Code workflows):

- **Concrete input/output examples** — the most effective fix when a prose spec is interpreted inconsistently across attempts.
- **Test-driven iteration** — write the test suite first (expected behavior, edge cases, performance), then iterate by feeding failures back.
- **The interview pattern** — have Claude ask clarifying questions *before* implementing in an unfamiliar domain (cache invalidation strategy? staleness tolerance? per-user or global?). Use for unfamiliar domains, non-obvious implications, multiple viable approaches.
- **Batching vs sequencing fixes** — issues that *interact* → one detailed message together; independent issues → sequential fixes. Questions hinge on exactly this distinction.

## 4.5 Validation and retry-with-feedback

Extract → validate (JSON Schema / Pydantic / business rules) → on failure, retry with the original document, the incorrect extraction, and the *specific* error: "Field 'total' = 150, but sum(line_items) = 145. Re-check."

**Retry helps when** the information is present but misformatted/misplaced/arithmetically inconsistent. **Retry cannot help when** the information is absent from the source or lives in a document never supplied — retrying only burns API calls. Recognizing which side a failure falls on is a recurring question shape.

**Pydantic's role:** structural validation (types, requiredness, enums) plus custom validators for business logic (sum equals total, start < end); on failure, construct the error message for the retry loop; can also generate the JSON schema for `tool_use` — one source of truth.

## 4.6 Self-correction patterns

- Extract `stated_total` **and** `calculated_total`; set `conflict_detected` when they disagree — internal contradictions surface automatically instead of silently propagating.
- Add a `detected_pattern` field to findings recording which construct triggered them — when developers dismiss findings, you can analyze which patterns generate false positives instead of guessing.

---

## Exam reflexes for this domain

- Schema fixes syntax, never semantics — semantic failures need validation + retry or self-correction.
- Inconsistent behavior across attempts → add examples, not more prose.
- False-positive complaints → tighten criteria with include/exclude lists + examples.
- Retry only repairs what's *present*; absent information can't be retried into existence.
- Required fields on optional data cause fabrication; nullable + "other"/"unclear" enums preserve honesty.
