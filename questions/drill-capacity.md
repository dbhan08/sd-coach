# Drill prompt bank — capacity estimation

15 prompts grouped by difficulty. Drill mode picks 5 per session, doesn't repeat within session.

Each prompt has a model-answer outline (key points only — Drill mode expands to 2–4 sentences when scoring).

---

## Easy

### 1. Small B2B SaaS write QPS
**Prompt:** Estimate write QPS for a small B2B SaaS with 50K MAU and ~10 actions/user/day.
**Key points:** 50K × 10 = 500K writes/day = ~6/sec average; peak 3× = ~18/sec. Sanity: trivial — single small DB instance.

### 2. Niche social app DAU
**Prompt:** A niche social app has 5M MAU. Typical engagement is "weekly user" — users open it about 1× per week. Estimate DAU.
**Key points:** Weekly user → DAU ≈ MAU/7 ≈ 700K. A more sticky app would hit MAU/3 (daily-ish). Stating the engagement assumption is the whole point.

### 3. Instagram photo upload rate (rough)
**Prompt:** Estimate photos uploaded per second on Instagram. Use rough industry numbers.
**Key points:** ~500M DAU, ~100M photos/day = ~1200/sec average; peak 3× = ~3500/sec. Sanity: matches reported 100M photos/day order of magnitude.

### 4. Corporate email storage
**Prompt:** Estimate raw storage for 1 year of all email at a 10K-employee company, average 50 emails sent + received per person per day, 50KB/email.
**Key points:** 10K × 50 × 365 = 180M emails × 50KB = 9 TB. Plus attachments could 10× this. Single beefy server fits raw; production scale is 10–100×.

### 5. Video-call egress bandwidth
**Prompt:** A video-call service has 100K concurrent calls at 720p (~1.5 Mbps per stream, 2 streams per call: in + out per participant, 2 participants minimum).
**Key points:** 100K calls × 2 participants × 1.5 Mbps egress = 300 Gbps. Sanity: that's a real-money bandwidth bill — multi-region distribution is essential.

### 6. Wikipedia view QPS
**Prompt:** Wikipedia gets ~18B pageviews/month. Estimate QPS.
**Key points:** 18B / 30 / 86400 = ~7000/sec average. Peak 2–3× = 15–20K/sec. Sanity: well within reach of a heavily-cached static-content fleet.

---

## Medium

### 7. Twitter 5-year storage
**Prompt:** Estimate storage for 5 years of every tweet at current scale (~500M tweets/day, average 200 bytes/tweet including metadata).
**Key points:** 500M × 200B = 100 GB/day × 1825 = 180 TB raw. Add 3× replication + indexes (~3× more) = ~1.5 PB effective. Sanity: that's ~150× a single 10-TB SSD, multi-rack but not heroic.

### 8. Media site CDN egress bill
**Prompt:** A media site has 100M DAU, each reads 5 articles/day, average page is 200KB after CDN compression. Estimate monthly CDN egress at $0.02/GB (committed-use rate).
**Key points:** 100M × 5 = 500M page-views/day × 200KB = 100 TB/day = 3 PB/month. At $0.02/GB → $60K/month. Sanity: list price ($0.05–0.08) would be 2.5–4× higher; commit pricing is real money saved.

### 9. Vertical search QPS
**Prompt:** A vertical search engine has 50M MAU, average user does 10 searches/day.
**Key points:** 50M / 30 (assuming 1/3 daily-active) → 17M DAU × 10 = 170M searches/day = ~2K/sec average; peak 3× = ~6K/sec. Sanity: that's a real search fleet — single instance won't do it.

### 10. Memcached cluster sizing for product cache
**Prompt:** You have a 1B-row product table, average row 1KB. You want 80% cache hit rate. Caching the head of the access distribution typically means caching ~20% of rows for 80% hits (Pareto).
**Key points:** 20% × 1B × 1KB = 200 GB working set. Round up to 256 GB; 4 cache nodes at 64 GB each for headroom + redundancy. Sanity: working set is much smaller than the full table — that's the whole point of caching.

### 11. Redis memory for chat tail
**Prompt:** WhatsApp-scale: 2B users, store the last 1000 messages of every conversation in Redis for fast read. Average user has 10 active conversations. Average message ~100 bytes.
**Key points:** 2B users × 10 conversations × 1000 messages × 100B = 2 PB. Sanity: that's enormous — production wouldn't actually keep all this hot. The lesson: "store the last N in cache" doesn't scale to billions of users without per-user TTLs or LRU.

### 12. S3 backup cost
**Prompt:** Photo backup service: 200M users, average 5 photos/week, 4MB each, 1-year retention. Cost at S3 standard $0.023/GB-month.
**Key points:** 200M × 5 × 52 × 4MB = 200 PB cumulative over the year. Average storage at 6 months = 100 PB. Cost: 100 PB × 1024 × 1024 GB × $0.023 ≈ $2.4M/month. Sanity: huge — Glacier tier ($0.004/GB) cuts to ~$400K/month.

---

## Hard

### 13. Globally-cached photo service egress bill
**Prompt:** 100M DAU × 30 photos/day average × 200KB compressed each. Globally cached. Estimate monthly egress bill at $0.05/GB list, then at typical commit-discount rate.
**Key points:** 100M × 30 × 200KB = 600 TB/day = 18 PB/month. At list: 18M GB × $0.05 = $900K/month. With CloudFront committed-use + cache offload (most served from edge): real bill 30–50% = $300–450K/month. Sanity: ridiculous numbers come from list pricing; nobody pays list at this scale.

### 14. Real-time stock-tick fan-out
**Prompt:** Service streams 5K stock symbols, each updates 1×/second. 100K subscribers, each subscribes to ~50 symbols. Estimate egress.
**Key points:** Each subscriber sees 50 updates/sec × 100B/update = 5 KB/sec/subscriber = 40 Kbps × 100K subs = 4 Gbps. Sanity: small for a fanout service. The interesting cost isn't egress — it's the per-symbol fan-out logic (5K source streams × ~2K subscribers each = 10M virtual streams to manage).

### 15. Tiered storage for security-camera footage
**Prompt:** 100K cameras × 1 Mbps × 90-day total retention, with 7 days on hot tier, 30 days on warm, 53 days on cold. Hot $0.10/GB-month, warm $0.025, cold $0.004.
**Key points:** Per camera: 1 Mbps × 86400 = 10.8 GB/day. 100K cameras × 10.8 GB = 1.08 PB/day ingest. Hot (7 days): 7.5 PB × $0.10 = $750K/month. Warm (30 days): 32.4 PB × $0.025 = $810K/month. Cold (53 days): 57 PB × $0.004 = $230K/month. Total ~$1.8M/month. Sanity: hot tier dominates; aggressive demote saves real money.
