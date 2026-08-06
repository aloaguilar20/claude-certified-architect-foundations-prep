# Domain 5 — Context Management & Reliability (15%)

Production concerns: error taxonomies and recovery, escalation to humans, keeping the context window useful over long runs, preserving provenance, and asynchronous batch processing.

**Official docs:** [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows) · [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing) · [Agent SDK hooks](https://code.claude.com/docs/en/agent-sdk/hooks)

---

## 5.1 Error categories

| Category | Examples | Retryable? | Action |
|---|---|---|---|
| Transient | Timeout, 503, network failure | Yes | Retry with exponential backoff |
| Validation | Bad input format, missing required field | No (fix input) | Modify request, retry |
| Business | Policy violation, threshold exceeded | No | Explain; propose an alternative |
| Permission | Access denied | No | Escalate |

### Anti-patterns

| Anti-pattern | Problem | Correct approach |
|---|---|---|
| Generic status ("search unavailable") | Coordinator can't choose a recovery | Return error type, query, partial results, alternatives |
| Silent suppression (empty result = success) | "No matches" and "search failed" become indistinguishable | Distinguish them explicitly |
| Aborting the whole workflow on one failure | All partial results lost | Continue with partials; annotate gaps |
| Infinite retries inside a subagent | Latency, wasted resources | 1–2 local retries, then propagate to the coordinator |

### Structured subagent errors

Give the coordinator enough to decide (retry with modified query? use partials? delegate elsewhere? annotate and continue?):

```json
{
  "status": "partial_failure",
  "failure_type": "timeout",
  "attempted_query": "AI impact on music industry 2024",
  "partial_results": [{"title": "AI Music Generation Report", "relevance": 0.8}],
  "alternative_approaches": ["Narrower query: 'AI music composition tools'"],
  "coverage_impact": "Not covered: AI impact on music production"
}
```

And carry gaps into the final output as **coverage annotations**: `### Music (PARTIAL COVERAGE — search agent timeout)`.

## 5.2 Escalation and human-in-the-loop

**Reliable triggers:**

| Situation | Action |
|---|---|
| Customer explicitly asks for a human | Escalate immediately — don't attempt to solve first |
| Policy doesn't cover the request (e.g. competitor price matching when policy is silent) | Escalate |
| No progress after reasonable attempts | Escalate |
| Financial operation above threshold | Escalate — enforced via hook, not prompt |
| Multiple matches on customer lookup | Ask for more identifiers; never guess |

**Unreliable triggers (named distractors):** sentiment analysis (mood ≠ complexity), model self-rated confidence 1–10 (confidently wrong; poorly calibrated), adding a trained classifier (over-engineering).

**Nuanced escalation:** first expression of frustration ≠ request for a human. Acknowledge the emotion → offer a concrete resolution → escalate when the customer *reiterates* the request.

**Structured handoff:** the human sees only your summary, not the transcript — it must be self-contained: customer/order IDs, issue summary, root cause, actions already taken, amounts, recommended action, escalation reason.

**Confidence calibration for extraction systems:** field-level confidence scores; thresholds tuned on labeled validation sets; high confidence → automated path, low confidence/ambiguous source → human review. **Stratified sampling:** audit samples even of high-confidence output — an aggregate 97% accuracy can hide 40% errors for one document type. Analyze accuracy by type and by field.

## 5.3 Context management in production

- **Fact blocks:** extract key facts (IDs, amounts, status) into a structured block re-included in *every* prompt — immune to lossy history summarization.
- **Trim tool results:** PostToolUse hook keeps the 5 relevant fields of a 40-field response.
- **Position-aware input:** key findings at the top, details in the middle, action items at the end (lost-in-the-middle mitigation).
- **Scratchpad files:** long investigations write key findings to a file; when context degrades (or in a new session) the agent consults the scratchpad instead of re-running discovery.
- **Subagent delegation:** an Explore-style subagent reads 15 files and returns one line; the coordinator acts as a separate context layer — aggregating outputs, holding global state, and giving each subagent a minimal context budget plus a constrained toolset (fewer tools = fewer distractions and lower context cost). Prevents context leakage between concerns.
- **Structured state persistence:** each agent exports state (`status`, `key_findings`, `coverage`, `gaps`) to a known location; a manifest lets the coordinator resume after a crash without redoing completed work.

## 5.4 Preserving provenance

- **Attribution loss:** "The market is estimated at $3.2B" is useless downstream. Keep claim + source URL/name + publication date + confidence together as a unit.
- **Conflicting data:** never arbitrarily pick one value. Preserve both with methodology and dates (`12%` per Spotify's 2024 report vs `8%` per a label survey), set `conflict_detected`, let the coordinator (or a human) decide.
- **Dates prevent false contradictions:** "A says 10%, B says 15% — contradiction" vs "A (2023) 10%, B (2024) 15% — likely growth."
- **Render by content type:** financial data → tables; analysis → prose; technical findings → structured lists; time series → chronological.

## 5.5 Message Batches API

([Batch processing docs](https://platform.claude.com/docs/en/build-with-claude/batch-processing))

| Attribute | Value |
|---|---|
| Cost | **50% of synchronous** pricing |
| Window | Most batches complete well under **24 hours**; no latency SLA |
| Correlation | `custom_id` links each request to its result |
| Shape | Each batch request is a single Messages API invocation. Tools and full conversation history are supported *in* a request — but executing a tool and returning its result requires a new request, so an interactive agent loop cannot complete inside a batch. |

**Batch vs synchronous:**

| Task | API | Why |
|---|---|---|
| Pre-merge PR check | Sync | A developer is blocked; up-to-24h is unacceptable |
| Overnight tech-debt report | Batch | Needed by morning; 50% savings |
| Weekly security audit | Batch | Not urgent |
| Interactive review | Sync | Immediate response required |
| 10,000 documents | Batch | Bulk; savings are significant |

**Failure handling:** identify failed items by `custom_id`, fix the strategy (e.g. chunk over-long documents), resubmit *only* the failures.

**SLA math:** result needed in 30h, batch takes up to 24h → submission must happen within the first 6 hours; submit no later than 24h before any deadline.

---

## Exam reflexes for this domain

- Errors carry category + retryability + attempted input + partials — always enough for the *next* decision.
- One subagent failing ≠ workflow abort; continue with partials and annotate coverage.
- Sentiment and self-rated confidence are never keyed escalation triggers.
- Numbers, dates and sources survive only if extracted into structure before summarization.
- Anything a human waits on stays synchronous; anything overnight/bulk goes to batch at half price.
