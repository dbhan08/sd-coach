# Nuro calibration

Nuro builds autonomous delivery vehicles — small, road-going AVs with no human inside. SWE infra roles at Nuro look more like "robotics + cloud + fleet ops" than web-shaped FAANG roles. This calibration biases toward edge/cloud split, real-time pipelines, fleet-scale telemetry, and safety-aware reasoning.

> Calibrated from the publicly visible shape of AV-company SWE work, not from confirmed Nuro interview rubrics. Treat as a reasoned default; revise after you sit a real loop.

## Question pool (L3–L4)

L3–L4 SWE-shaped problems. Avoid the perception/planning ML internals — those are specialized roles, not general SWE.

- Design a **fleet telemetry ingestion** service: vehicles emit structured events + sensor metadata; cloud must ingest, store cheaply, and make searchable for offline review. Bandwidth is constrained.
- Design a **mission/order dispatch** service: incoming delivery orders are matched to available vehicles in a service area, with ETA estimates and reassignment when a vehicle drops out.
- Design **OTA software updates** for the fleet: roll out a new on-vehicle build to N thousand vehicles with staged rollout, health gating, and rollback.
- Design a **vehicle health monitoring** service: each vehicle reports component health on a heartbeat; cloud detects anomalies and pulls vehicles out of service before a fault becomes unsafe.
- Design **on-vehicle log capture + selective upload**: drives produce TB of sensor data per day; only a small slice (interesting events, anomalies) gets uploaded for offline review. Design the trigger + upload path.
- Design a **simulation scenario library + replay infra**: store driving scenarios extracted from real drives, let an offline simulator pull them, run them against new on-vehicle builds, and report regressions.
- Design a **remote assistance / teleop request** routing service: a vehicle stuck on something it can't decide pings cloud; cloud routes to an available human operator within latency budget.

Avoid: full perception stack, full motion planner, HD-map authoring pipeline (all Staff+ or specialized).

## Rubric weights

- **weight up:**
  - `rubrics/hld.md → scoping` (Nuro questions almost always have an edge/cloud split — naming what runs where, and what's safety-critical vs not, is half the answer)
  - `rubrics/hld.md → data-flow` (real-time vs batch, edge vs cloud, lossy vs lossless paths — naming each path matters)
  - `rubrics/hld.md → tradeoff-articulation` (bandwidth vs fidelity, autonomy vs cloud-dependency, latency vs cost — every Nuro design has these)
  - `rubrics/lld.md → error-handling` (vehicle-side failures must fail safe, not just fail loud)
  - `rubrics/lld.md → observability` (every event matters for safety review and post-incident analysis)
  - `rubrics/capacity-estimation.md → assumptions` (the bandwidth/storage numbers from sensor data are wildly different from web — get the assumptions right and the design follows)
- **weight down:**
  - `rubrics/hld.md → communication` (Nuro engineers tend to value precision and substance over interview-polish narration)

## Cultural emphasis

**Safety-aware, not safety theater.** The expectation isn't that you recite ISO 26262 or claim safety expertise — it's that you can identify *which parts of your design are safety-critical* and treat them differently. Cloud-side analytics? Not safety-critical, fail loudly is fine. On-vehicle decision to keep driving when telemetry upload fails? Safety-critical, must fail safely (vehicle keeps operating, telemetry gets buffered, no cascading dependency on cloud). Naming this split unprompted is a strong signal.

**Edge-first thinking.** A common L3–L4 miss is designing the cloud pipeline first and then "adding the vehicle" as an afterthought. Better Nuro answers start at the vehicle: what does the vehicle need to do autonomously, what does it actually need from cloud (and on what timescale), what happens when the link is degraded or gone. Then design cloud to that shape.

**Bandwidth is real.** A single AV can produce multi-TB/day of raw sensor data. You cannot stream all of it. Designs that implicitly assume "and then we send it to the cloud" without naming bandwidth, sampling, on-vehicle filtering, or trigger-based upload are missing the central constraint of AV infra. Conversely, naming "we'd compute features on-vehicle and only ship the structured event stream + a small video clip on trigger" is a strong move.

**Fleet-shaped, not single-vehicle-shaped.** Most problems shift character between 10 vehicles and 10,000. A design that "works for one vehicle and scales linearly" is usually wrong somewhere — fan-in, hot regions, dispatch contention, OTA rollout safety. Acknowledging fleet scale as a first-class dimension matters.

## Anti-patterns (flag explicitly)

- **Sync coupling vehicle → cloud on a safety-critical path.** "The vehicle queries cloud for the next maneuver" — hard miss. Vehicles must operate autonomously; cloud is for fleet coordination, telemetry, and slow-loop decisions, not real-time control.
- **Streaming raw sensor data to cloud without acknowledging bandwidth.** "All the LiDAR + camera frames go to S3" without naming TB/day, on-vehicle filtering, or trigger-based upload is a flag.
- **No story for a vehicle going offline.** LTE/5G drop. The vehicle must keep being a vehicle. Designs that assume always-on connectivity are wrong.
- **Treating telemetry as "just logs."** Telemetry at an AV company is the substrate for safety review, regression detection, and incident analysis. "We'd put it in CloudWatch" misses what telemetry is *for* in this context.
- **Unbounded blast radius on OTA.** Pushing a new build to the whole fleet at once is a recall waiting to happen. Staged rollout + health gating + automatic rollback should be named, not asked-for.
- **Conflating "real-time" tiers.** Vehicle control loop (10s of ms), teleop (~100ms), telemetry ingest (seconds), offline replay (minutes-hours) are wildly different problems. Designs that paper over the timescales are weaker than ones that name them.

## L3 vs L4 expectation

**L3 (entry):**
Identify the edge/cloud split and name what runs where. Acknowledge bandwidth as a real constraint when sensor data is in scope. Get one safety-critical-vs-not distinction stated. Capacity numbers within an order of magnitude. Name at least one failure mode of the link between vehicle and cloud and how the vehicle behaves through it.

**L4 (mid):**
Same as L3 plus: explicit reasoning about timescales (control loop vs telemetry vs offline). On-vehicle filtering / trigger-based upload as a first-class design move, not an afterthought. For OTA-flavored questions, staged rollout + health gating + rollback unprompted. For dispatch-flavored questions, what happens when a vehicle drops mid-mission. At least one place where the design accepts data loss for bandwidth/cost — and names what's lost.

## Notes

- Public stack signals at AV companies generally: ROS / ROS 2 (or in-house equivalents) for on-vehicle messaging, protobuf for schemas, time-series DBs (InfluxDB / VictoriaMetrics / proprietary) for vehicle telemetry, object storage for log/clip archival, Kafka or similar for event streaming, custom HD map formats. You don't need to name these — reasoning about systems with their properties is what's tested.
- Nuro specifically operates small low-speed delivery AVs (no passengers, public roads). The "no human inside" framing changes the failure-mode conversation: the vehicle stopping safely is almost always an option, which is different from passenger AVs.
- If the user mentions a JD that's heavier on cloud/backend than on robotics, lean the question pool toward fleet/dispatch/OTA and away from on-vehicle log capture.
