# Drill prompt bank — component design

15 prompts grouped by difficulty. Each: design one small component end-to-end in ~3 minutes — interface, primitive, one tradeoff.

---

## Easy

### 1. Web session store
**Prompt:** Design a session store for a web app. Single-region, ≤100K active sessions, sessions expire after 24h of inactivity.
**Key points:** Redis with TTL = 24h sliding (refresh on read). Key = session_id, value = {user_id, created_at, last_seen}. Tradeoff: in-memory means no durability across Redis restart — fine for sessions (user just re-auths). At 100K × ~200 bytes = 20 MB, single instance is overkill.

### 2. JWT validator
**Prompt:** Design the auth token validator that runs at every API gateway request. Tokens are JWTs.
**Key points:** Local validation: parse JWT, verify signature against cached public key (rotated daily), check exp claim. No DB call required — JWT is self-contained. Tradeoff: revocation is hard (have to wait for token to expire OR maintain a revocation list / short-TTL tokens with refresh).

### 3. Tiny KV store
**Prompt:** Design a key-value store API for ≤10K keys, single-process, in-memory, persists on shutdown.
**Key points:** Hash map in process. On shutdown, serialize to file. On start, load file. Operations: get, put, delete. Tradeoff: process crash loses recent writes — fine for cache-like uses, not OK for durable state. Mitigation: log-and-replay on every put (turns this into a basic embedded DB).

### 4. Buffered logger
**Prompt:** Design a logging library that buffers writes and flushes on size threshold (1MB) or time threshold (5s).
**Key points:** In-memory ring buffer with two flush triggers. Background flusher checks the time. Public API: `log(level, msg)` — non-blocking (fire-and-forget into the buffer). Tradeoff: process crash loses up to 5s/1MB of logs. For audit logs, flush synchronously instead.

### 5. Read-heavy counter
**Prompt:** Design a counter that's read 1000× per second and written 1× per second, single-region.
**Key points:** Single Redis INCR + GET. Or even a single in-memory atomic int with periodic snapshot for durability. Tradeoff: at this read/write asymmetry, caching is basically free — no need for a more complex design.

---

## Medium

### 6. Global rate limiter
**Prompt:** Per-user rate limiter for a public API: 100 requests/minute, enforced globally across regions.
**Key points:** Token bucket per user, stored in Redis. Atomic Lua script: read current tokens + last-refill timestamp, compute new tokens based on elapsed time, decrement, write back. Tradeoff: cross-region accuracy requires a globally-distributed Redis (cost, latency) OR per-region buckets that drift slightly (acceptable for most rate limits). Fail-open if Redis is down.

### 7. URL shortener (no analytics)
**Prompt:** Design just the shorten + redirect paths for a URL shortener. 100M URLs/month, 10× clicks. No analytics required.
**Key points:** POST /shorten generates random 7-char base62 code, INSERTs into Postgres with UNIQUE constraint, retries on collision. GET /:code reads from Postgres (or hot cache for popular codes), returns 301. Tradeoff: random codes vs. hash-of-URL — hash gives idempotency at the cost of not being able to mask the long URL.

### 8. Image upload service
**Prompt:** Design an image upload service for 1MB–10MB files at 1K writes/sec.
**Key points:** Pre-signed URL pattern: client requests upload URL from API, gets a signed S3 PUT URL, uploads directly to S3 (offloads bandwidth from API server). Metadata (filename, owner, S3 key) in Postgres. Tradeoff: client is trusted to actually upload — must verify on read OR run a background job to delete orphan metadata.

### 9. Notification dedup
**Prompt:** Ensure a user doesn't receive the same notification twice in a 24-hour window.
**Key points:** Redis SET with key = `dedup:<user_id>:<notification_hash>`, value = timestamp, TTL = 24h. On notification send, SETNX (set-if-not-exists) — if it returns 1, send; if 0, skip. Tradeoff: hashing the notification matters — too coarse and you suppress different notifications; too fine and you don't dedup.

### 10. Distributed work queue
**Prompt:** Design a queue for jobs that take 10 seconds to 10 minutes, with automatic retry on failure (up to 3 attempts).
**Key points:** SQS / RabbitMQ with visibility timeout = 11 minutes (longer than max job). Worker pulls a job, marks invisible while processing, ACKs on success or NACKs on failure. Job message includes attempt count; after 3 failures, moves to dead-letter queue. Tradeoff: visibility-timeout-based retry can double-deliver if a worker is slow but not failed — jobs must be idempotent.

---

## Hard

### 11. Viral-post like counter
**Prompt:** Distributed counter for likes on a viral post. 50K writes/sec sustained. Eventually-consistent reads OK. Hot-partition aware.
**Key points:** Sharded counter — each post's counter is split across N=100 sub-counters. Write lands on a random sub-counter (no coordination needed). Read sums all sub-counters (parallelizable). Tradeoff: read amplification (100× the work of a single read), but eliminates the hot-partition problem entirely. Combine with periodic rollup to a single counter for cheap reads of less-hot posts.

### 12. Real-time presence service
**Prompt:** "Which of your friends are online?" service for 1M concurrent users, 200 friends average.
**Key points:** Each user maintains a heartbeat (e.g., every 30s) that sets `presence:<user_id>` in Redis with TTL 60s. Online = key exists. To get a user's online friends, read their friend list, MGET all the presence keys. Tradeoff: 200 keys × 1M users querying continuously = a lot of Redis ops; alternative is push-based with a pub/sub fanout to socket connections, much more efficient at scale.

### 13. Idempotent payment-charging service
**Prompt:** Charge a credit card. Must be safely retryable — if the response is lost, retrying must not double-charge.
**Key points:** Client supplies an Idempotency-Key header (UUID). Server stores `(idempotency_key, request_hash, response)` in a transactional store. On request: if key exists with matching hash, return cached response; if key exists with different hash, reject as 409. If key doesn't exist, take a row-lock, perform the charge, store the result, release. Tradeoff: state required for as long as retries are possible (often 24h+) — adds storage cost but is non-negotiable for payments.

### 14. Distributed lock service
**Prompt:** Short-lived (≤30s) leader election among 100 worker nodes.
**Key points:** etcd or ZooKeeper ephemeral nodes — workers race to create `/lock/<resource>`, holder owns the lock, ephemeral node disappears when worker disconnects (heartbeat lost). For Redis Redlock: similar, simpler, but has known clock-skew correctness issues — fine for non-critical locks. Tradeoff: etcd/ZK has correctness; Redis has performance + simplicity but isn't a true consensus system.

### 15. Feature flag service
**Prompt:** Queried 100K QPS by 1000 services. Must work even when central service is down.
**Key points:** Push model: central service publishes flag changes to a Pub/Sub topic; each service subscribes and caches the flag set in-memory. Reads = local lookup, no central call. On startup, fetch the full flag set with a fallback to a baked-in default file. Tradeoff: flag changes propagate with delay (seconds), but the system is robust to central outages — exactly what the prompt requires.
