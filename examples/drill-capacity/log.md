# Drill — capacity — synthetic example

**Synthetic — illustrative only.** Not a real run.

Company emphasis: none
Start: 2026-05-07T16:00:00

---

### Rep 1 — medium

**Prompt:** Estimate write QPS for a global photo-sharing app with 200M DAU, where each user uploads on average 0.5 photos per day.

**Answer:** "200M × 0.5 = 100M writes/day. 100M / 86400 ≈ 1160 writes/sec average. Apply a 3× peak factor for evenings → ~3500 peak writes/sec."

**Model:** 200M × 0.5 = 100M writes/day = 1160/sec average. Peak factor 2–5× depending on geographic distribution and event timing — call it 3500–5800 peak. Sanity check: 3500 photos/sec is well within a single sharded write tier; not at all the bottleneck for this scale (storage + read fanout will be).

**Score:** **hit** — assumptions stated implicitly, math right, peak factor applied.

---

### Rep 2 — medium

**Prompt:** Estimate storage needed for 5 years of every Tweet at current Twitter scale (assume 500M tweets/day, average 200 bytes/tweet including metadata).

**Answer:** "500M × 200 bytes = 100 GB/day. 5 years × 365 = 1825 days. 100 × 1825 = 182,500 GB ≈ 180 TB."

**Model:** 500M × 200 bytes = 100 GB/day. 5 years ≈ 1800 days. 100 × 1800 = 180 TB. Sanity check: that's about 18 modern SSDs at 10 TB each — easily fits in a single rack. Tweets are tiny; it's the indexing and replicas that make Twitter's storage real (3× replication + multiple indexed views = 3–10× the raw number).

**Score:** **hit** — math right, units consistent, sanity-checked implicitly with the SSD reference. Missed the "but with replication and indexes it's bigger" bonus.

---

### Rep 3 — hard *(stepped up after 2 consecutive hits)*

**Prompt:** Estimate egress bandwidth bill in USD/month at $0.05/GB for a globally-cached photo service serving 100M DAU each viewing 30 photos/day average, photos average 200 KB compressed.

**Answer:** "100M × 30 = 3B views/day. × 200KB = 600,000 GB/day = 600 TB/day. × 30 days = 18,000 TB/month = 18 PB/month. At $0.05/GB → $900K/month? That seems too high."

**Model:** 100M × 30 × 200KB = 600 TB/day = 18 PB/month. At $0.05/GB egress that's 18M GB × $0.05 = $900K/month. Sanity check: AWS egress at scale is heavily discounted (committed-use can be 1¢/GB or less), and a CDN like CloudFront passes most of that egress through at lower rates AND offloads from origin. Real bill is maybe 30–50% of this number. The number being "too high" is the right instinct — this is why nobody pays list price for egress at this volume.

**Score:** **partial** — math correct, but the candidate flagged "too high" as a problem instead of investigating *why* (commit discounts, CDN economics). The instinct was right; the follow-through was missing.

---

### Rep 4 — hard

**Prompt:** A messaging service has 50M MAU. Average user sends 30 messages/day. Each message is replicated 3× for durability and stored for 1 year. Average message is 100 bytes plus 50 bytes of metadata. Estimate raw storage needed.

**Answer:** "50M × 30 = 1.5B messages/day. × 365 = 547B messages/year. × 150 bytes = 82 TB. × 3 replicas = 246 TB."

**Model:** 50M × 30 × 365 × 150 bytes ≈ 82 TB raw, × 3 replicas = ~250 TB. Sanity check: 250 TB / 10 TB SSD = 25 SSDs. That's a small rack. Messaging at this scale is dominated by index + read amplification (per-conversation indexes, full-text search if any, push-notification delivery state) — the raw storage is the floor, not the ceiling.

**Score:** **hit** — math and units right, included replication factor. Missed the "raw vs effective" distinction but rep is timed; that's L4-hit-not-perfect territory.

---

### Rep 5 — hard

**Prompt:** A real-time multiplayer game has 1M concurrent users in 100K matches (10 players each on average). Each client sends 30 state updates per second. Estimate the bandwidth a single game server handles if it's responsible for 1000 matches.

**Answer:** "1000 matches × 10 players = 10K clients per server. Each client sends 30 updates/sec → 300K incoming msg/sec. Each update is small, say 100 bytes → 30 MB/sec ingress. Outgoing is the same fanout — server broadcasts each player's state to the other 9 players, so 30 × 9 = 270 outgoing msg/sec per client, × 10K clients = 2.7M outgoing msg/sec × 100 bytes = 270 MB/sec. Sanity check: 270 MB/sec is ~2.2 Gbps, that's pushing a single server."

**Score:** **hit** — clean fan-out reasoning, applied to both incoming and outgoing, sanity-checked against a real network bandwidth reference. This is L4-territory work.

---

## End: 2026-05-07T16:18:00

## Tally
- Hits: 4
- Partials: 1
- Misses: 0

## Trend
Strong on math and applying peak/replication factors. Sanity-checking is present but the sanity-check led to a "this seems too high" deadend in rep 3 instead of digging into why (commit discounts, CDN economics) — the instinct was right; the follow-through was missing. Difficulty stepped up after 2 hits and stayed at hard for the rest of the drill.
