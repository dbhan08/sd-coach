# Scorecard — URL shortener — synthetic example

**Synthetic — illustrative only.** Not a real run.

Target: L3–L4 SWE / SDE1–SDE2
Company calibration: none

## Per-dimension scores

### HLD (rubrics/hld.md)

- **scoping** [L3 hit, L4 partial]
  - rubrics/hld.md → scoping: "Out of scope: spam protection, brand safety. Does that match?" | Restated scope clearly with explicit in/out, but didn't name an NFR (latency target, availability target). For L4, name at least one — e.g., "p99 read latency target 100ms."

- **component-breakdown** [L4 hit]
  - rubrics/hld.md → component-breakdown: "Three components for the MVP. First, an API server... Second, a Postgres DB... Third, a CDN or HTTP cache..." | Clean three-component split, each with single responsibility, no overlap.

- **data-flow** [L4 hit]
  - rubrics/hld.md → data-flow: "URLs in this system are immutable — you never re-shorten an existing code — so we don't need invalidation." | Walked both read and write paths, named the cache asymmetry, articulated *why* invalidation isn't needed.

- **scaling** [L3 hit, L4 partial]
  - rubrics/hld.md → scaling: "Most clicks come from a small head of viral links." | Identified head/tail distribution as motivation for cache, but didn't quantify (what's the hit rate target? what fraction of links are head?). For L4, attach numbers to the intuition.

- **tradeoff-articulation** [L4 hit]
  - rubrics/hld.md → tradeoff-articulation: "Tradeoff: at much higher scale I'd switch to a KV store and handle collision in the app layer with retries." | Named the alternative (KV store) and the cost (lose unique-constraint enforcement) and the threshold to switch.

- **communication** [L4 hit]
  - rubrics/hld.md → communication: (consistent throughout) | Drove the conversation, signposted phase transitions, recovered from the idempotency tension without prompting.

### Capacity (rubrics/capacity-estimation.md)

- **assumptions** [L3 hit, L4 partial]
  - rubrics/capacity-estimation.md → assumptions: "Let me state assumptions. 100M new URLs/month..." | Assumptions explicit, plausible. Didn't identify which assumption is load-bearing (the 100B-bytes/record one drives the storage decision; perturbation analysis was implicit).

- **math** [L4 hit]
  - rubrics/capacity-estimation.md → math: "100M / 30 / 86400 ≈ 40 writes/sec average... With a 3× peak factor, peak read QPS is ~1.2K." | Math right, units explicit, peak/average distinction applied.

- **sanity-checks** [L3 hit]
  - rubrics/capacity-estimation.md → sanity-checks: "1.2 TB. That fits on a single beefy node — sanity check, that's reasonable for a URL shortener." | Sanity-checked storage against a real-world reference (single node).

- **relevance** [L4 hit]
  - rubrics/capacity-estimation.md → relevance: storage estimate directly drove the "single Postgres node, no sharding needed for MVP" decision.

### LLD (rubrics/lld.md)
*(Phase 4 captured here; phases 5–6 elided in this synthetic example.)*

- **interface-design** [L4 hit]
  - rubrics/lld.md → interface-design: "Should be idempotent — same long URL submitted twice should... by my generation strategy it'll get a new code. Could add idempotency via an Idempotency-Key header if a client wants stable behavior." | Named idempotency tension, stated the design choice, and named the mitigation.

## Highlights
1. Clean tradeoff articulation — every nontrivial decision named what's given up.
2. Capacity estimation drove design (storage → no sharding) instead of being a ritual.
3. Recognized the cache invalidation question is a non-issue for this design and explained why.

## Growth areas
1. Scoping: name an explicit NFR (latency/availability target) before designing.
2. Scaling: when you cite a head/tail distribution, attach numbers (target hit rate, % of head).
3. Capacity: identify which assumption is load-bearing, not just which ones are stated.

## Verdict
- **L3 bar:** clear — scoping, math, primitive choices all hit.
- **L4 bar:** partial — strong on tradeoffs, communication, and component breakdown; light on NFR specification and quantified scaling intuitions.
- **To clear L4:**
  1. State at least one explicit NFR (latency or availability target) during scoping.
  2. When citing read/write asymmetry, attach a number (cache hit rate target, head/tail split estimate).

## Deferred concepts
*(none in this run)*
