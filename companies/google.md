# Google calibration

## Question pool (L3–L4)

- Design a URL shortener (TinyURL) — the canonical Google L3–L4 prompt.
- Design a global rate limiter for an API gateway (per-user, per-API-key, per-IP).
- Design a web crawler that respects robots.txt, deduplicates URLs, and processes a fixed seed set with bounded memory.
- Design a typeahead/autocomplete service for search queries.
- Design a distributed counter for a "likes" feature that needs to hit eventually-consistent storage with reasonable accuracy.
- Design Google Drive's basic file upload + sync — focus on the upload path, conflict resolution as a follow-up.
- Design a key-value store on top of unreliable nodes (introductory consensus / replication discussion).

Avoid: full Spanner/Bigtable internals (Staff+), full Maps/YouTube infra (too broad).

## Rubric weights

- **weight up:**
  - `rubrics/capacity-estimation.md → assumptions` (Google interviewers explicitly reward stated assumptions before math)
  - `rubrics/capacity-estimation.md → math` (arithmetic correctness is non-optional)
  - `rubrics/hld.md → component-breakdown` (clean boundaries, not a smear)
  - `rubrics/hld.md → tradeoff-articulation` (what you give up should be named, every time)
- **weight down:**
  - `rubrics/hld.md → communication` (Google interviewers tolerate slower, more methodical answers — substance over polish)

## Cultural emphasis

Google interviews push on **fundamentals and rigor**. Sloppy capacity estimation is a flag. "I'd use Redis" without saying *why* Redis and not Memcached/Cassandra/etcd is a flag. Tradeoffs are expected on every nontrivial decision — not as a ritual ("CAP theorem!") but as a real cost-benefit articulated for *this* system.

The SRE framing matters. Talk about SLOs, error budgets, latency tails (p99, p99.9), not just averages. "The p50 is 100ms" is half an answer; "the p99 is 500ms because of GC pauses on the JVM cache, which we'd fix by warming N replicas" is a Google-shaped answer.

Google interviewers care about whether you can reason from first principles. Knowing "Spanner uses TrueTime" is fine, but reasoning about why a globally-consistent DB needs synchronized clocks (or some equivalent) is what gets credit. Don't lean on jargon if you can't unpack it.

## Anti-patterns (flag explicitly)

- **Hand-wavy capacity.** "We'd cache it" is not a capacity answer. "Read QPS is 50K, p99 read latency target is 100ms, so we need a hot cache that fits 80% of the working set in memory — that's about 200GB across 4 nodes" is a capacity answer.
- **Missing the read/write ratio.** Almost every Google design hinges on this. Skipping the question is a flag.
- **Wrong primitive without justification.** Saying "I'd use a relational database" for a workload that's clearly a wide-column or KV access pattern, with no acknowledgment, gets dinged.
- **Treating "scale" as a magic word.** Just saying "we'd shard" without saying *what's the shard key*, *what happens to range queries*, *what about hot shards* is hollow.
- **Ignoring tail latency.** Google specifically interviews for p99/p99.9 thinking, not just averages.

## L3 vs L4 expectation

**L3 (entry):**
State assumptions before estimating. Get the math within an order of magnitude. Name the right primitive and one alternative you considered. Identify one obvious tradeoff (consistency vs availability, latency vs cost). Get the headline capacity numbers right.

**L4 (mid):**
Same as L3 plus: discuss tail latency explicitly. Reason about hot shards / hot keys, not just average load. Name at least one failure mode of your chosen primitive and the mitigation. For the schema, justify indexes against the access pattern. For the API, name idempotency and what happens on retry.

## Notes

- Google's preferred stack (internally): Borg/Kubernetes, gRPC, Bigtable/Spanner, GFS/Colossus, Pub/Sub. You don't have to use these names, but reasoning about systems with their properties is what's tested.
- Capacity questions at Google are not optional. Even if the prompt is "design a URL shortener," they will steer to "how big is your storage, how many QPS."
