# HLD rubric

High-level design rubric. Modes cite dimensions from this file when scoring (e.g., `rubrics/hld.md → scaling`).

This is the skeletal version. Session 7 expands each dimension with explicit L3–L4 line items, common traps, and "what would be needed to clear L5."

## Dimensions

### scoping
Did the candidate establish what's in/out, who the users are, what scale we're targeting, and which features are MVP vs nice-to-have *before* drawing boxes? Score reflects clarity of the contract being designed against, not the depth of the questions.

### component-breakdown
Are the major services / boundaries identified and justified? Does each component have a clear single responsibility, or are concerns smeared across boxes? Does the candidate know why these are separate services and not one?

### data-flow
For the headline use cases, can you trace the request from client to storage and back? Are read paths and write paths distinguished? Does the candidate know where the latency actually lives?

### scaling
How does the design handle the stated scale? Sharding strategy, caching layer, async vs sync boundaries, where the bottleneck moves at 10×. Did the candidate say "we'd cache" or did they say what to cache, where, with what invalidation?

### tradeoff-articulation
When the candidate picked SQL over NoSQL (or push over pull, or sync replication over async), did they name what they're giving up? Generic "it depends" is a miss; concrete "we lose X to gain Y" is a hit.

### communication
Structure, signposting, recovery from blocks. Did the candidate drive the conversation or wait to be driven? When stuck, did they articulate what they're stuck on?

## Citation format

When a mode references this rubric in feedback or a scorecard, use:

```
rubrics/hld.md → <dimension>: <verbatim quote of the candidate's relevant answer> | <one-sentence judgment>
```

Example:
> rubrics/hld.md → scaling: "I'd use Redis for caching" | Generic — didn't specify what to cache (read-through? hot keys? full denorm view?), how to invalidate, or what the cache hit rate target is.
