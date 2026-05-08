# Capacity estimation rubric

Back-of-envelope rubric. Modes cite as `rubrics/capacity-estimation.md → <dimension>`.

Calibrated for **L3–L4 SWE / SDE1–SDE2** roles. The L3 bar is "you can do the math when asked"; L4 is "you bring it up before being asked, and your numbers drive design decisions."

## assumptions

**What it measures:** Did the candidate state the inputs (DAU, MAU, ratio of writes to reads, average payload size, retention) *before* calculating?

- **L3 hit:** Stated 3+ assumptions explicitly before doing math. Assumptions are plausible for the named system (not "100M DAU for a small B2B SaaS").
- **L3 partial:** Some assumptions stated, others snuck in mid-math without being called out.
- **L3 miss:** Math done with no stated assumptions, OR assumptions wildly implausible for the system shape.
- **L4 hit:** L3 hit + acknowledged at least one assumption that's load-bearing for the design AND said what would change if that assumption is off by 10×.
- **L4 partial:** L3 hit but assumptions are stated mechanically without identifying which ones matter.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Reaching for "1B users" because it's a recognizable number. Match the assumption to the product.
- Plausible-sounding numbers that don't match each other (claiming 10K writes/sec but 100M DAU — the math doesn't work).
- Treating the company name as a scale (Twitter ≠ "10B QPS" — you have to actually estimate).

**Clears L5+:** Picks assumptions that drive the design tradeoff being discussed, not generic numbers. Identifies which assumption is the *uncertain* one and structures the design around it.

## math

**What it measures:** Is the arithmetic right? Are units consistent (per-second vs per-day vs per-year)? Does QPS = DAU × actions/day / 86400 actually get computed?

- **L3 hit:** Math is right within an order of magnitude. Units are consistent. The candidate showed their work (didn't just state a number).
- **L3 partial:** Math has an error of one order of magnitude that the candidate didn't catch, OR units quietly drift mid-calculation.
- **L3 miss:** Math is off by 2+ orders of magnitude, OR refused to commit a number.
- **L4 hit:** L3 hit + applied a peak-to-average factor (or explicitly noted that they're estimating peak) AND the units are explicit at every step.
- **L4 partial:** L3 hit but no peak/average distinction.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- 86,400 seconds per day vs 100,000 — pick a number for mental math (I use 100K, knowing I'm 16% high; that's fine).
- Missing the peak-to-average factor (peak QPS is 2–5× average for most workloads).
- Confusing per-user-per-second with system-wide-per-second.

**Clears L5+:** Math that's not just right but *useful* — the numbers map directly to a design decision (how many cache nodes, how big a fleet, how much storage budget).

## sanity-checks

**What it measures:** After getting a number (e.g., "2 PB of storage"), did the candidate sanity-check it?

- **L3 hit:** After landing a number, said one of: "is that plausible?", "how does that compare to <reference>?", "does that fit on N machines?". Did the check at least once.
- **L3 partial:** Stated the number, didn't sanity-check, but the number happens to be plausible.
- **L3 miss:** Stated the number, didn't sanity-check, AND the number is implausible.
- **L4 hit:** L3 hit + sanity-checked every load-bearing number, not just one. At least one sanity check leveraged a real-world reference (a single SSD is ~10 TB; a single node typically handles ~10–50K QPS for simple workloads).
- **L4 partial:** L3 hit on some numbers, missed on others.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Confidence in a number with no check. "200 PB" might be right or might be off by 1000×.
- Using the result as if it's exact when it's an order-of-magnitude estimate.
- Not knowing reference values (1 TB SSD ≈ $100, 1 GB DRAM ≈ $5, 1 GB egress ≈ $0.05–$0.10) — these change but having rough order-of-magnitude saves you here.

**Clears L5+:** Sanity-checks against multiple references (storage, compute, network, dollar cost). Catches their own errors before the interviewer does.

## growth

**What it measures:** Does the design handle 10× growth without re-architecting, or does it bake in assumptions that break at scale?

- **L3 hit:** Stated explicitly where the design's capacity envelope is. ("This works up to ~5× current scale; past that, the single-region DB becomes the bottleneck.")
- **L3 partial:** Vague "we could scale" without naming the breaking point.
- **L3 miss:** Treated current scale as static.
- **L4 hit:** L3 hit + named the *specific component* that fails first at 10× AND has a sketched-out path for what to do then (shard the DB, add a cache layer, switch to async, etc).
- **L4 partial:** L3 hit but the path forward at 10× is hand-wavy.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Designing for current scale, claiming it scales, never showing how.
- Designing for hypothetical scale, ignoring current operational cost.
- Assuming sharding "just works" without naming the shard key.

**Clears L5+:** Names the bottleneck at 10×, the bottleneck at 100×, AND the architectural cost of moving from one to the other. Reasons about whether to design for it now or later.

## relevance

**What it measures:** Is the candidate estimating the things that *matter* for the design decision, or generating numbers to look thorough?

- **L3 hit:** The numbers estimated map to at least one design decision in the HLD (write QPS → DB choice; bandwidth → CDN sizing; storage → retention policy).
- **L3 partial:** Some numbers map to decisions, some are decorative.
- **L3 miss:** Numbers are decorative — the design would be the same regardless of what was estimated.
- **L4 hit:** L3 hit + the candidate explicitly said "this number is what makes me pick X" at least once, AND skipped estimating a quantity that wasn't decisive ("we don't need to estimate egress bandwidth here because it's not on the critical path").
- **L4 partial:** L3 hit but every decision gets every number, regardless of relevance.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Estimating MAU, DAU, sessions/day, time-on-app, and ad impressions per user when only write QPS matters.
- Estimating without a follow-through to a design decision.
- Treating estimation as a ritual instead of a tool.

**Clears L5+:** Estimates ruthlessly — only the numbers that drive a decision, with explicit "this is why this number matters."

## Citation format

```
rubrics/capacity-estimation.md → <dimension>: "<quote>" | <judgment> [L3/L4 hit | partial | miss]
```
