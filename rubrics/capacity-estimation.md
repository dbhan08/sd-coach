# Capacity estimation rubric

Back-of-envelope rubric. Modes cite as `rubrics/capacity-estimation.md → <dimension>`.

Skeletal — Session 7 adds worked examples and common traps per dimension.

## Dimensions

### assumptions
Did the candidate state the inputs (DAU, MAU, ratio of writes to reads, average payload size, retention) *before* calculating? Are the assumptions plausible for the named system? Saying "let's assume 100M DAU" for a small B2B SaaS is a miss — the number must match the product.

### math
Is the arithmetic right? Are units consistent (per-second vs per-day vs per-year)? Is the candidate avoiding off-by-orders-of-magnitude errors? Does QPS = DAU × actions/day / 86400 (with peak-to-average factor) actually get computed?

### sanity-checks
After getting a number (e.g., "2 PB of storage"), did the candidate sanity-check it? "Is 2 PB plausible? That's [X] disks worth — yes/no." A raw number with no sanity check is a soft miss even if the math is right.

### growth
Does the design handle 10× growth without re-architecting, or does it bake in assumptions that break at scale? Did the candidate state where the design's "designed for" envelope ends?

### relevance
Is the candidate estimating the things that *matter* for the design decision (e.g., write QPS to choose a database; bandwidth to size CDN), or just generating numbers to look thorough? Estimating is a tool, not a ritual.

## Citation format

```
rubrics/capacity-estimation.md → <dimension>: <quote> | <judgment>
```
