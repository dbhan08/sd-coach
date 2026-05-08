# HLD rubric

High-level design rubric. Modes cite dimensions from this file when scoring (e.g., `rubrics/hld.md → scaling`).

Calibrated for **L3–L4 SWE / SDE1–SDE2** roles. The L3 bar is what an entry-level hire should clear; L4 is what's expected of a mid-level engineer who has shipped real services. "Clears L5+" notes are for awareness, not targets — modes do not score against L5.

## scoping

**What it measures:** Did the candidate establish what's in/out, who the users are, what scale we're targeting, and which features are MVP vs nice-to-have *before* drawing boxes? Score reflects clarity of the contract, not the depth of the questions.

- **L3 hit:** Asked at least 3 of: who are the users, what are the headline features, what scale are we targeting, what's explicitly out of scope. Restated the scope in their own words before moving on.
- **L3 partial:** Asked some scoping questions but jumped to design before the contract was clear, OR scoping was implicit and never restated.
- **L3 miss:** Started designing before any scoping. Treated the prompt as fully specified.
- **L4 hit:** L3 hit + named at least one explicit non-functional requirement (latency target, availability target, consistency requirement) AND identified the trickiest part of the problem in scoping.
- **L4 partial:** L3 hit but no NFR / no signal on what's interesting about the problem.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Asking many questions but never landing on a contract before designing.
- Asking only scale questions; missing user intent.
- Assuming the prompt is fully specified ("Design Twitter" → diving in).

**Clears L5+:** Drives the scoping conversation, suggests the right NFRs based on product shape, identifies and articulates the *non-obvious* constraint (e.g., "for messaging, the latency target is human-perception — under 200ms feels real-time; that's our budget across the read path").

## component-breakdown

**What it measures:** Are the major services / boundaries identified and justified? Does each component have a clear single responsibility?

- **L3 hit:** Named 4–7 major components for a typical L3 prompt. Each has a clear name and a one-sentence responsibility. No two components have overlapping responsibility.
- **L3 partial:** Components named but at least one has muddled responsibility ("API server that also does analytics") or two near-duplicates.
- **L3 miss:** A monolith with no internal boundaries, OR boxes drawn with no clear responsibility per box.
- **L4 hit:** L3 hit + each boundary is justified (why is this its own service? team ownership? failure isolation? scaling profile?). At least one component's *non-presence* is justified ("we don't need a separate notification service yet because…").
- **L4 partial:** L3 hit but boundaries are unjustified or "because microservices are good."
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Splitting into services purely along CRUD lines (UserService, PostService, …) with no deeper reason.
- Premature SOA — "we'll have 12 microservices on day one" for a small system.
- Conflating logical concerns (auth, rate limiting, caching) with deployment units.

**Clears L5+:** Justifies each boundary by deployment lifecycle, failure isolation, scaling profile, AND team ownership. Articulates which components could be merged in early-stage and split later.

## data-flow

**What it measures:** For the headline use cases, can you trace the request from client to storage and back? Are read paths and write paths distinguished?

- **L3 hit:** Walked through the headline write path AND headline read path step by step. Named where the data lives at each step. Identified the hot path.
- **L3 partial:** Walked one path (usually write) but the read path was hand-waved or symmetric to write.
- **L3 miss:** Could not trace a request through the components without prompting.
- **L4 hit:** L3 hit + named the latency contributors at each hop AND identified at least one place where read path differs structurally from write path (cache, denormalized view, async pipeline).
- **L4 partial:** L3 hit, but didn't name latency contributors or didn't acknowledge read/write asymmetry.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Treating reads and writes symmetrically when the system is read-heavy or write-heavy.
- Not knowing where the latency lives ("I'd cache it" without saying what gets cached, where, when it's invalidated).
- Skipping the failure path — what happens when the cache is cold, the queue is full, the DB is down.

**Clears L5+:** Reasons about p99 latency at each hop, identifies the contention points before being asked, and walks the failure path proactively.

## scaling

**What it measures:** How does the design handle the stated scale? Sharding strategy, caching layer, async vs sync boundaries, where the bottleneck moves at 10×.

- **L3 hit:** Named at least one specific scaling primitive (sharding by X, caching at Y with Z TTL, async queue between A and B) AND said why it's the right primitive for *this* system.
- **L3 partial:** Said "we'd cache" or "we'd shard" without naming what or how.
- **L3 miss:** No scaling thinking. Single-instance assumption.
- **L4 hit:** L3 hit + identified where the bottleneck moves at 10× growth AND named at least one place where scale forces a tradeoff in product behavior (e.g., "feed becomes 30s stale instead of real-time at this scale").
- **L4 partial:** Generic 10× answer without specifics; OR specifics without acknowledging the product cost.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Conflating "use Redis" with "we have a caching strategy."
- Sharding without naming the shard key or thinking about hot shards.
- Adding a queue everywhere without thinking about ordering or back-pressure.

**Clears L5+:** Reasons about the cost model (dollars + ops), identifies the next-bottleneck-after-the-current-fix, and reasons about cross-region scaling explicitly.

## tradeoff-articulation

**What it measures:** When the candidate picks SQL over NoSQL (or push over pull, or sync replication over async), do they name what they're giving up?

- **L3 hit:** On at least 2 nontrivial decisions, said *what* they're trading and *why* it's worth it. ("Sync replication for the primary user record because we can't tolerate inconsistency on auth; async for the analytics stream because lag is fine.")
- **L3 partial:** Said "tradeoff" but didn't name what's lost. Generic "it depends" with no concrete cost.
- **L3 miss:** Made decisions with no acknowledgment of alternatives or costs.
- **L4 hit:** L3 hit on every nontrivial decision (not just 2), AND for at least one decision named the alternative they considered and rejected.
- **L4 partial:** L3 hit but tradeoffs are stated, not reasoned (recite-mode).
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Citing "CAP theorem" without grounding it in what the user sees.
- Picking a primitive and never naming the cost (every primitive has a cost).
- Naming a tradeoff without the cost being concrete ("eventually consistent" — over what window? what does the user see during it?).

**Clears L5+:** Articulates which tradeoffs are actually decisive vs ritualistic, and pushes back on the interview prompt's implicit constraints when they're worth pushing on.

## communication

**What it measures:** Structure, signposting, recovery from blocks. Did the candidate drive or wait to be driven? When stuck, did they articulate what they're stuck on?

- **L3 hit:** Drove the conversation. Signposted phase transitions ("OK let me move to the API now"). When stuck, said what they were stuck on instead of going silent.
- **L3 partial:** Drove most of the time but had stretches of silence or hand-waving when uncertain.
- **L3 miss:** Required the interviewer to pull the conversation forward at every step.
- **L4 hit:** L3 hit + recovered from a wrong direction without the interviewer having to redirect (recognized the issue, named it, course-corrected).
- **L4 partial:** L3 hit but needed redirection from the interviewer at least once.
- **L4 miss:** Same as L3 miss.

**Common traps (L3–L4):**
- Long silences when uncertain. Better to think out loud or ask a clarifying question.
- Switching topics mid-thought. "Oh wait, also for the cache —" without finishing the previous decision.
- Reciting answers without engaging with the specific question.

**Clears L5+:** Communication is so clear that the interviewer's job is mostly listening. Drives both the technical content AND the structure of the conversation.

## Citation format

When a mode references this rubric in feedback or a scorecard, use:

```
rubrics/hld.md → <dimension>: "<verbatim candidate quote>" | <one-sentence judgment>
```

Example:
> rubrics/hld.md → scaling: "I'd use Redis for caching" | Generic — didn't specify what to cache (read-through? hot keys? full denorm view?), how to invalidate, or what the cache hit rate target is. **L3 partial.**
