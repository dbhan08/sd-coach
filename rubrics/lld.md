# LLD rubric

Low-level design rubric for the critical-path module the candidate is asked to drill into. Modes cite as `rubrics/lld.md → <dimension>`.

Calibrated for **L3–L4 SWE / SDE1–SDE2** roles.

## interface-design

**What it measures:** Are the public methods/endpoints minimal, well-named, and orthogonal? Are inputs/outputs typed? Are error cases explicit?

- **L3 hit:** Defined 3–6 public methods/endpoints. Each has a name that describes what it does. Inputs and outputs are typed (or schema'd). At least one error case is explicit (return type or status code).
- **L3 partial:** Methods defined but vague names (`process()`), missing types, or error cases implicit ("returns null on failure" — when?).
- **L3 miss:** No interface defined or interface is "we'll have an API."
- **L4 hit:** L3 hit + idempotency is named where it matters (POST that should be safe to retry). At least one method's behavior is specified for partial failure (input validated, half-applied — what's the contract?).
- **L4 partial:** L3 hit but no idempotency thinking, or unclear partial-failure contract.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Methods that take a "context" or "options" blob instead of typed parameters.
- Missing pagination on list endpoints (any list endpoint at any scale needs paging — interview red flag).
- Verbs as nouns or vice versa (`/users/create` vs `/createUser` — pick a style and stick to it).

**Clears L5+:** Designed for evolution — versioning strategy, deprecation path, schema-additive change patterns.

## data-model

**What it measures:** Schema for the chosen module: tables, indexes, constraints, denormalization choices.

- **L3 hit:** Defined the primary entity with its fields, primary key, and at least one secondary index. Stated what queries the schema is supporting.
- **L3 partial:** Schema defined but indexes missing or unjustified, OR queries the schema supports are unstated.
- **L3 miss:** No schema, OR schema that doesn't match the access pattern.
- **L4 hit:** L3 hit + named at least one denormalization choice with its motivation (read amplification of normalized form), AND named one constraint that has to be enforced at the application layer because the DB can't.
- **L4 partial:** L3 hit but no denormalization thinking; OR denormalization without naming the cost (write amplification, consistency).
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Indexes added without thinking about write cost.
- "JSON column for everything" without acknowledging the queryability cost.
- Missing the obvious uniqueness constraint (email, username, slug).

**Clears L5+:** Reasons about row size, hot partitions, write amplification, and tail-latency on indexed writes.

## error-handling

**What it measures:** What happens when the upstream is slow, when a dependency 500s, when a write partially succeeds, when input is malformed?

- **L3 hit:** Distinguished user errors (4xx — return clearly) from system errors (5xx — log + maybe retry) from transient errors (retry with backoff). Named at least one specific case.
- **L3 partial:** "We'd handle errors" without specifics, OR conflated user/system/transient.
- **L3 miss:** No error story.
- **L4 hit:** L3 hit + retries are bounded AND idempotent at the right layer (saying "we retry" without idempotency is a flag). Named timeout strategy explicitly. Acknowledged at least one cascading failure mode.
- **L4 partial:** L3 hit but retries unbounded or non-idempotent, OR no timeout discipline.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Unbounded retries on 5xx ("we'd retry until it works" — until you DDoS yourself).
- No timeouts on outbound calls (any sync call without a timeout will eventually wedge the caller).
- Retry on non-idempotent operations (POST that's not idempotent + retry = duplicate creates).

**Clears L5+:** Reasons about retry budgets, exponential backoff with jitter, and circuit breakers as a system property, not a library.

## concurrency

**What it measures:** Is there shared state? Is access to it serialized correctly?

- **L3 hit:** Identified the shared state and named the synchronization mechanism (lock, queue, optimistic concurrency, single-writer pattern). Recognized at least one race condition possibility.
- **L3 partial:** Mentioned "we'd handle concurrency" without specifics.
- **L3 miss:** No concurrency thinking despite the design having clear shared state.
- **L4 hit:** L3 hit + named the cost of the chosen synchronization (lock contention, queue depth, ABA problem) AND has a story for the case where the synchronization fails (lock holder crashes, etc).
- **L4 partial:** L3 hit but no cost reasoning.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Reaching for a distributed lock when partitioning would eliminate the contention.
- Optimistic concurrency without an explicit conflict-detection field (version, ETag, timestamp).
- Assuming "the database handles it" without naming what isolation level is in play.

**Clears L5+:** Reasons about contention at scale, lock-free alternatives where appropriate, and tradeoffs between strict serializability and weaker isolation.

## observability

**What it measures:** What does the candidate log, count, and trace? Can you tell from telemetry alone whether this module is healthy?

- **L3 hit:** Named at least one metric per request (latency, error rate). Named what to log on error (with correlation/request ID). Identified at least one structured event worth recording.
- **L3 partial:** Said "we'd log" without saying what, OR named metrics that aren't actionable.
- **L3 miss:** No observability story.
- **L4 hit:** L3 hit + RED (Rate, Errors, Duration) or USE (Utilization, Saturation, Errors) metric set is identifiable, AND there's a story for debugging a single failed request via tracing or correlation IDs.
- **L4 partial:** L3 hit but metrics are scattershot, no clear way to debug a single failure.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Logging everything, alerting on nothing.
- Dashboards but no SLOs.
- Using counts when distributions are needed (latency averaged is a lie; p99 is the truth).

**Clears L5+:** Designs SLOs that match the user-experienced contract, not arbitrary thresholds; reasons about cardinality cost in metrics and structured-log storage.

## Citation format

```
rubrics/lld.md → <dimension>: "<quote>" | <judgment> [L3/L4 hit | partial | miss]
```
