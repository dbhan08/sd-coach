# Scorecard — Nuro fleet telemetry — synthetic example

**Synthetic — illustrative only.** Not a real run.

Target: L3–L4 SWE / SDE1–SDE2
Company calibration: Nuro (`companies/nuro.md`)

## Per-dimension scores

### HLD (rubrics/hld.md, weighted per Nuro calibration)

- **scoping** [L4 hit] — *weighted up*
  - rubrics/hld.md → scoping: "telemetry upload is *not* on any safety path. The vehicle keeps driving regardless..." | Led with the safety-critical split unprompted, stated the "vehicle never blocks on telemetry" NFR explicitly, named in/out clearly. This is exactly what the Nuro calibration weights up — hit on the first signal.

- **component-breakdown** [L4 hit]
  - rubrics/hld.md → component-breakdown: "On-vehicle, three components. A ring buffer... A trigger detector... And a telemetry agent..." | Clean component split on both sides of the edge/cloud boundary. Each component has a single responsibility. The "gateway off the clip data path" decision shows component boundaries chosen for a reason, not by default.

- **data-flow** [L4 hit] — *weighted up*
  - rubrics/hld.md → data-flow: walked structured path and clip path separately and named *why* they diverge (validation matters for one, throughput dominates the other). | The Nuro calibration weights this up because real-time vs batch and edge vs cloud paths are the central data-flow distinctions in fleet telemetry — the candidate hit both.

- **scaling** [L3 hit, L4 partial]
  - rubrics/hld.md → scaling: Kafka retention sized to indexer outage tolerance was a sharp move. | Didn't explicitly address fleet growth — what happens at 10K vehicles instead of 1K, where does the design first break? For L4 at Nuro, fleet-shaped scaling reasoning is expected.

- **tradeoff-articulation** [L4 hit] — *weighted up*
  - rubrics/hld.md → tradeoff-articulation: "we accept structured-event loss in extended-outage scenarios to keep the vehicle's on-disk footprint bounded, and we keep clips because they're the high-value records." | Named the lossy class AND the durable class AND why. This is the L4 bar at Nuro — accepts loss for a constraint and names what's lost. Hit.

- **communication** [L3 hit]
  - rubrics/hld.md → communication: signposted phase transitions, drove the conversation. | Adequate — Nuro calibration weights this *down*, so this dimension isn't doing much work in the verdict.

### Capacity (rubrics/capacity-estimation.md, weighted per Nuro calibration)

- **assumptions** [L4 hit] — *weighted up*
  - rubrics/capacity-estimation.md → assumptions: "the *load-bearing* assumption is the events/sec figure — if it's 1000/sec/vehicle instead of 100, structured ingest is 40 TB/day and the design changes." | Identified the load-bearing assumption explicitly. This is the L4 differentiator on this dimension and the candidate hit it cleanly.

- **math** [L3 hit, L4 partial]
  - rubrics/capacity-estimation.md → math: arithmetic right within order of magnitude, units explicit. | Could have shown the peak-vs-average decomposition (1000 vehicles isn't all driving at once — what's the peak fleet-active fraction?). For L4, that distinction tightens the ingest sizing.

- **sanity-checks** [L4 hit]
  - rubrics/capacity-estimation.md → sanity-checks: "10 TB/day is manageable for object storage but is a real bill... Worth flagging because it makes retention policy a real design question, not an afterthought." | Sanity check fed back into the design (retention as a first-class question) rather than being a pro-forma "yeah that's a lot." That's the L4 move.

- **relevance** [L4 hit]
  - rubrics/capacity-estimation.md → relevance: bandwidth/storage estimate motivated the gateway-off-clip-path decision and the retention sizing — capacity drove design.

### LLD (rubrics/lld.md)
*(Phases 4–6 elided in this synthetic example.)*

## Nuro-specific signal

From `companies/nuro.md → cultural emphasis`:

- **safety-aware:** *strong hit.* Led with the safety-critical-vs-not split unprompted. Restated "vehicle never blocks on telemetry" multiple times. Named that telemetry is not on any safety path.
- **edge-first thinking:** *strong hit.* Started at the vehicle and worked outward — "since the vehicle constraints drive most of the design."
- **bandwidth is real:** *hit.* TB/day numbers, on-vehicle filtering via triggers, signed-URL upload to keep gateway off the bytes path.
- **fleet-shaped:** *partial.* Designed for 1K vehicles cleanly but didn't address what changes at 10K — see scaling growth area.

From `companies/nuro.md → anti-patterns`: none triggered. Notably did *not*:
- Treat raw sensor data as something to stream to cloud (used triggers + ring buffer)
- Couple the vehicle synchronously to cloud
- Skip the offline / vehicle-offline story (queues, bounded loss)

## Highlights
1. Safety-critical vs not split surfaced unprompted in scoping and held throughout — the strongest Nuro-shaped signal in the run.
2. Lossiness named as a deliberate, asymmetric design choice (events lossy, clips durable) with the *why* attached.
3. Capacity drove design — the cost sanity-check fed back into retention policy as a real design lever.

## Growth areas
1. **Fleet-shaped scaling.** Address what changes at 10K vehicles — where does this design first break? Hot regional gateway? Kafka partitioning? Index write rate?
2. **Peak vs average on the ingest math.** 1000 vehicles aren't all driving simultaneously; the peak-active fraction tightens the sizing and changes the gateway capacity decision.
3. **Schema evolution story.** Telemetry schemas change as the on-vehicle stack evolves; how do old vehicles emitting an old schema coexist with new consumers? Worth naming for L4.

## Verdict
- **L3 bar:** clear — safety split, edge-first reasoning, capacity assumptions, tradeoffs all hit.
- **L4 bar:** clear on the Nuro-weighted dimensions (scoping, data-flow, tradeoff-articulation, assumptions). Partial on scaling and math.
- **To tighten further:**
  1. Address fleet-growth scaling explicitly during HLD.
  2. Decompose peak vs average in the capacity phase.

## Deferred concepts
*(none in this run)*
