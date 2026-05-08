# Netflix calibration

## Question pool (L3–L4)

- Design a notification service that fans out push notifications to N million subscribers when a new episode drops.
- Design a per-user "Continue Watching" tracker — durable across devices, eventually-consistent reads OK.
- Design a service that ingests playback heartbeats from clients and decides when to mark a title as "watched."
- Design a feature flag service that handles 100K services querying flags at high QPS, with graceful degradation if the central service is down.
- Design the read path for the Netflix homepage: rows of recommendations, personalized per user, must work even when downstream services are partially unavailable.
- Design a service that schedules/triggers transcoding jobs for newly uploaded titles. (Single trigger source, large fan-out to compute.)

Avoid: full Open Connect / CDN design (Staff-level), full recommendations ML system (out of scope for L3–L4).

## Rubric weights

- **weight up:**
  - `rubrics/hld.md → scaling` (Netflix scale is implicit in every question)
  - `rubrics/hld.md → tradeoff-articulation` (especially around availability vs consistency)
  - `rubrics/lld.md → error-handling` (Netflix culture is "things will fail — what then?")
  - `rubrics/capacity-estimation.md → growth` (assume 10× isn't optional)
- **weight down:**
  - `rubrics/lld.md → concurrency` (less frequently a focus than at Google/Meta)

## Cultural emphasis

Netflix is **availability-first**. Every part of the system should keep working when something downstream fails. Talk about graceful degradation, fallbacks, circuit breakers, and what the user sees during partial outages — not just steady-state performance.

Chaos Engineering matters here. Saying "we'd add retries" without thinking about the cascade (retry storms, thundering herds) is a miss. A strong answer names the failure mode AND the mitigation AND what gets sacrificed (e.g., "fall back to cached recommendations from yesterday — user sees stale rows but no error").

Microservice boundaries should be justified by **failure isolation** as much as by team ownership. "These are separate services so a failure in X doesn't bring down Y" is a Netflix-shaped answer.

## Anti-patterns (flag explicitly)

- **Sync coupling on the user-facing path.** Any user-facing read that synchronously depends on N microservices, all of which need to be up — Netflix specifically engineered around this with Hystrix-era circuit breakers. Calling this out earns points; ignoring it is a flag.
- **"Eventually consistent is fine"** without saying what the user sees during the inconsistency window. (Be specific: "user might see yesterday's continue-watching row for ~30s after switching devices — that's acceptable for this UX.")
- **No fallback story.** Designing the happy path with no answer to "what happens if Cassandra is down?" is a hard miss.
- **Treating this like a single-region problem.** Netflix is multi-region active-active; a design that assumes one region is leaving signal on the table.

## L3 vs L4 expectation

**L3 (entry):**
Identify the obvious failure modes. Name circuit breakers, timeouts, retries with backoff. Articulate at least one fallback for the user-facing path. Get the capacity numbers within an order of magnitude.

**L4 (mid):**
Same as L3 plus: explicit retry-storm reasoning, named cache TTLs and what's behind the cache when it expires, and a defensible answer for cross-region behavior on at least the read path. Should also flag at least one place where the design accepts staleness for availability — and name what's stale.

## Notes

- Netflix uses Cassandra, Kafka, EVCache (Memcached fork), and historically Eureka + Hystrix. Mentioning these by name is fine but shouldn't be a substitute for reasoning. Saying "I'd use Cassandra" is weaker than "I'd want a wide-column store for the time-series access pattern, Cassandra fits."
