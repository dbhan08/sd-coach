# Mock — Nuro fleet telemetry — synthetic example

**Synthetic — illustrative only.** Not a real run.

Company: Nuro
Target: L3–L4
Start: 2026-05-08T10:00:00

---

## Phase 1: Scoping

**Interviewer:** Today's problem: design fleet telemetry ingestion for Nuro's delivery vehicles. Vehicles emit data while operating; cloud needs to ingest it, store it, and make it searchable for offline review by safety and engineering. You have 45 minutes. Ask anything before you start.

**Candidate:** OK. First — there's an obvious split here. The vehicle is real-time and safety-critical; cloud is not. So I want to be explicit: telemetry upload is *not* on any safety path. The vehicle keeps driving regardless of whether telemetry is reaching cloud. Is that consistent with how you're framing it?

**Interviewer:** Yes. Telemetry is for offline review, not control.

**Candidate:** Good. Then a few questions. Roughly what fleet size — tens, hundreds, thousands? What kinds of events — structured events only, or sensor data too? And what's the "searchable for offline review" use case — engineers triaging an incident, or batch analytics, or both?

**Interviewer:** Assume ~1000 vehicles in this design, growing. Both kinds of data: a high-rate structured event stream, and on-trigger clips of recent sensor data — short snippets when something interesting happens, like a hard brake or a planner exception. Both engineer triage and batch analytics matter.

**Candidate:** Got it. So scope: the vehicle emits two streams — a structured event stream at high rate, and trigger-driven clips of recent sensor data. Cloud ingests, stores, indexes, and exposes for query. In scope: ingest path, storage layout, indexing for search, basic backpressure / loss handling. Out of scope: the on-vehicle perception stack that *generates* events, the alerting UI, the offline analytics jobs themselves. NFR: structured events queryable within ~10 seconds for ops dashboards; clips within minutes; vehicle never blocks on telemetry.

**Interviewer:** Match. Move on.

[note: candidate led with the safety-critical split unprompted — `companies/nuro.md → cultural emphasis: safety-aware`: hit. Stated NFRs explicitly including the "vehicle never blocks" property — `rubrics/hld.md → scoping`: L4 hit.]

## Phase 2: Capacity estimation

**Interviewer:** Numbers.

**Candidate:** Let me state assumptions. 1000 vehicles, each operating ~10 hours/day. Structured event rate: assume ~100 events/sec/vehicle when active — that's a guess; could be 50 or 200, but order of magnitude. Average event size: ~500 bytes structured (protobuf-shaped). So per-vehicle structured rate is ~50 KB/sec when driving. Across the fleet at full activity that's 50 MB/sec ingress, or ~4 TB/day of structured telemetry.

For clips: triggers are rare per vehicle — maybe 50/day per vehicle on average. Each clip is, say, 30 seconds of compressed sensor data — ballpark 100 MB. So 50 × 100 MB = 5 GB/day/vehicle, × 1000 = 5 TB/day of clip data. Total ingest ~10 TB/day.

Sanity check: 10 TB/day is manageable for object storage but is a real bill — at S3 list prices that's a few hundred dollars/day in storage growth alone, before egress for analytics. Worth flagging because it makes retention policy a real design question, not an afterthought. And the *load-bearing* assumption is the events/sec figure — if it's 1000/sec/vehicle instead of 100, structured ingest is 40 TB/day and the design changes.

**Interviewer:** Good. HLD.

[note: stated assumptions, identified the load-bearing one explicitly — `rubrics/capacity-estimation.md → assumptions`: L4 hit. Math right within order of magnitude — `→ math`: L3 hit. Sanity-checked against cost which feeds back into design — `→ relevance`: L4 hit. Bandwidth/storage as a real constraint surfaced unprompted — `companies/nuro.md → cultural emphasis: bandwidth is real`: hit.]

## Phase 3: HLD

**Interviewer:** Walk me through it.

**Candidate:** [draws hld.mmd] I'll start at the vehicle and work toward cloud, since the vehicle constraints drive most of the design.

On-vehicle, three components. A ring buffer holding the last ~5 minutes of raw sensor data — fixed-size, on-disk, oldest data evicted. A trigger detector watching the structured event bus — hard-brake, planner-exception, and similar predicates fire a snapshot of the ring buffer into a 30-second clip with surrounding context. And a telemetry agent that maintains two queues — one for the structured event stream, one for clips — and uploads over mTLS HTTPS when connectivity allows.

Critical property: the agent's queues are bounded. If cloud is unreachable for a long stretch, oldest structured events drop. Clips are higher-priority — they're rarer and triggered, so we keep them longer. The vehicle never blocks on upload. This is the lossiness I'm naming as a deliberate choice: we accept structured-event loss in extended-outage scenarios to keep the vehicle's on-disk footprint bounded, and we keep clips because they're the high-value records.

Cloud side. Regional ingest gateway terminates mTLS, validates schema, and fans out: structured events into Kafka, clips uploaded directly to object storage via signed URL (gateway issues the URL, the agent does the upload — keeps the gateway off the data path for big payloads).

Three downstream consumers off Kafka. A real-time consumer that powers fleet-ops alerting — only specific event classes, not every event. A batch sink into a columnar warehouse for offline analytics. And an indexer that feeds a search index — event metadata plus clip object keys — so an engineer triaging an incident can query "all events for vehicle X between 14:02 and 14:05" and pull the clips.

**Interviewer:** Why is the ingest gateway off the clip data path?

**Candidate:** Two reasons. One, clips are 100 MB; routing them through an app-tier gateway means the gateway needs to handle multi-TB/day throughput, which is wasteful — object storage handles that natively. Two, signed URLs let me move the auth check up front and the bytes go straight to S3, so a gateway outage doesn't block clip uploads that already have a URL. The structured stream stays through the gateway because schema validation matters more for that path — bad-shape events poison the warehouse.

**Interviewer:** What if the search indexer falls behind?

**Candidate:** The index is a derived view; Kafka is the source of truth with a retention window. If the indexer falls behind, queries return stale results, but no data is lost — when the indexer catches up, the events show up. The constraint is the Kafka retention window: if the indexer is down longer than that, we lose the ability to backfill and have to replay from the warehouse. So I'd size Kafka retention to the longest indexer outage we want to tolerate without warehouse replay — call it 7 days as a starting point.

**Interviewer:** OK. Onto failure modes.

[note: led at the vehicle and worked outward — `companies/nuro.md → cultural emphasis: edge-first thinking`: hit. Named lossiness as a deliberate design choice with the data class that's lossy AND the data class that isn't — `rubrics/hld.md → tradeoff-articulation`: L4 hit. Three-component split each with single responsibility — `→ component-breakdown`: L4 hit. Walked the structured path and the clip path separately and named *why* they diverge — `→ data-flow`: L4 hit. Recovered cleanly on the indexer-falls-behind question by naming Kafka as source of truth and tying retention to the SLO — strong on durability reasoning. The "vehicle never blocks on upload" property restated multiple times — strong on the safety-critical-vs-not split.]

---

*[remaining phases (failure modes, schema, LLD, observability) elided for brevity in this synthetic example]*

---

## Deferred questions
- (none in this run)

## End: 2026-05-08T10:42:00
