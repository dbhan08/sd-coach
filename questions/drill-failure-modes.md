# Drill prompt bank — failure modes

15 prompts grouped by difficulty. Each: given a system + a failure event, walk the cascade — what breaks, in what order, what does the user see first.

---

## Easy

### 1. Postgres primary down
**Prompt:** API server fronted by ALB, backed by single Postgres primary. Postgres goes down for 30 seconds. What does the user see, and at what timing?
**Key points:** T+0: writes start failing immediately, reads from connection pool return 5xx within 100ms. T+5s: connection pool exhausts, all requests fail fast. T+30s: Postgres comes back, pool reconnects, traffic recovers within seconds. User sees ~30s of "something went wrong" errors with no graceful degradation. **Mitigation:** read replica, async write queue for non-critical writes.

### 2. Service B returning 500s, no retry on A
**Prompt:** Service A calls Service B sync. B starts returning 500s for 50% of requests. A has no retry logic. What does the user see?
**Key points:** Linear: 50% of A's responses become 5xx, 50% succeed. Better than retry storm, worse than retry-with-backoff. User sees inconsistent, page-refresh-fixes-it errors. **Mitigation:** retry with exp backoff + jitter, capped at 2–3 attempts.

### 3. Browser localStorage cleared mid-session
**Prompt:** Web app stores session token in localStorage. User clears browser storage mid-session. What happens to in-flight requests?
**Key points:** Already-sent requests complete (their headers were already serialized). New requests have no token, get 401. Frontend ideally catches 401, redirects to login. User sees an unexpected logout. **Mitigation:** server-side session cookies (HttpOnly) instead of localStorage tokens — clearing storage no longer logs you out.

### 4. Expired TLS cert
**Prompt:** Mobile app polls a status API every 30 seconds. The API server's TLS cert expires at midnight. Walk the next 5 minutes.
**Key points:** T+0 (midnight): every TLS handshake fails. App polls fail with cert errors. T+30s, T+60s, ...: each polling cycle generates new failed handshakes. User sees "can't connect" or stale data. **Mitigation:** automated cert rotation (Let's Encrypt + monitoring), alerting on cert expiry T-30 days.

### 5. Redis OOM eviction
**Prompt:** Single Redis cache, no replica. Redis runs out of memory and starts evicting (LRU). What's the first user-visible symptom?
**Key points:** Cache hit rate drops; backend (DB) load spikes; latency increases; some operations that depended on cached state (sessions, rate limit counters) become inconsistent. First user-visible: slower responses, possibly 5xx if backend gets overloaded. **Mitigation:** alert on memory pressure, evict-on-write-back, scale up before OOM.

---

## Medium

### 6. No timeout on sync call
**Prompt:** Service A calls B sync over HTTP. B's p99 latency jumps from 100ms to 5s. A has no timeout configured. Walk the next 60 seconds across A's traffic.
**Key points:** T+0: A's request handlers start blocking 5s/request instead of 100ms. T+5s: A's thread/connection pool fills with stuck callers. T+10s: A starts rejecting incoming connections (pool exhausted). T+30s: A's upstream load balancer flags A as unhealthy and removes from rotation — *now* the cascade hits A's caller. User sees: a few seconds of slow, then total unavailability. **Mitigation:** sub-second timeouts on all outbound calls, circuit breaker.

### 7. Async replica auto-promote
**Prompt:** Postgres primary fails. Async replica auto-promotes — but it's 30 seconds behind. What does the user see in the next 60 seconds?
**Key points:** T+0: primary dies, writes fail. T+5–10s: failover detected, replica promoted. T+10s onward: writes accepted on new primary, BUT the most recent 30s of data is gone. Users who wrote in the last 30s before failure see their data missing — and may now be writing inconsistent state on top. **Mitigation:** sync replication for the last writes (cost: latency on every write), or app-level acknowledgment of "data may be lost on failover."

### 8. CDN serving stale due to origin 503
**Prompt:** CDN's origin is your API server. Origin starts returning 503s. CDN starts serving stale cached content (configured behavior). What does the user see?
**Key points:** Cached requests: stale-but-OK content (probably acceptable). Uncached requests: 503 from CDN. Mixed UX — some users see fine, some see errors. Worse: stale-while-revalidate may be returning stale data that's *wrong* now (e.g., updated prices, deleted items). **Mitigation:** stale-while-revalidate is fine for *static* content; for dynamic content, fail with a clear error rather than serve stale.

### 9. Kafka consumer crash + rebalance
**Prompt:** Kafka consumer group has 10 consumers on 12 partitions. One consumer crashes. Walk the rebalance.
**Key points:** T+0: 1 consumer crashes; its 1–2 partitions stop being processed. T+15s (heartbeat timeout): broker detects, triggers group rebalance. T+15–30s: ALL consumers stop and re-coordinate (the "stop the world" rebalance). T+30s: rebalance complete, partitions redistributed across remaining 9 consumers. Backlog accumulated during rebalance. User-visible: delay in any user-facing event consumed by this group. **Mitigation:** static membership (Kafka 2.3+) reduces rebalance frequency.

### 10. DNS misroute for 5% of clients
**Prompt:** DNS for your service's API endpoint suddenly returns the wrong IP for 5% of clients (DNS poisoning at one resolver chain). Walk the next 10 minutes.
**Key points:** T+0: 5% of clients connect to wrong IP, get connection refused or wrong service. Their app retries: same wrong IP (cached). T+TTL (e.g., 5 minutes): cache expires, they re-resolve, may or may not get the right IP. User-visible: 5% of users see persistent errors; refreshing doesn't fix. **Mitigation:** short TTLs on critical service DNS, monitoring DNS resolution from multiple geographic vantage points, DNSSEC.

---

## Hard

### 11. Cache eviction storm
**Prompt:** Read-heavy service, Redis cache + Postgres. Cache flushes (config push at 09:00:00 evicts everything). Walk through 09:01:00 — what fails, what slows, what recovers.
**Key points:** T+0: cache empty, every request hits Postgres. T+1s: Postgres CPU spikes to 100%, query latency goes from 1ms to 1s+. T+5s: client timeouts start firing, retries pile on, Postgres queue grows. T+10s: Postgres rejects new connections (max_connections hit). T+30s: backend errors propagate to user. T+60s: cache slowly fills as queries complete; load drops. Recovery is slow because every cold-cache miss adds load. **Mitigation:** warming on deploy, request coalescing (singleflight pattern) so 1000 concurrent identical requests = 1 backend call.

### 12. Retry storm without backoff
**Prompt:** Service A retries on 5xx with no backoff or circuit breaker. B is overloaded and slow. Walk the cascade.
**Key points:** T+0: B's latency rises, B starts returning 5xx. T+1s: A retries every failed request immediately. Effective load on B = 2–3× original. T+2s: B's queue fills, latency rises further, more 5xx. T+5s: A retries again, load = 5–10× original. T+10s: B is fully wedged, A's caller pool is exhausted (every request going through 3 retries). T+30s: cascading collapse, full outage. **Mitigation:** exponential backoff with jitter, retry budgets (e.g., max 10% of requests are retries), circuit breaker.

### 13. Cross-region partition
**Prompt:** Multi-region active-active service, with eventual-consistency replication. Network partition between us-east and us-west for 2 minutes. What happens during, what's the recovery?
**Key points:** During: each region serves writes locally (active-active). Reads from one region don't see writes from the other. Conflicting writes possible (same record updated in both regions). Recovery (T+2min): partition heals, replication catches up. Conflicts resolved via LWW (last-write-wins) or CRDT or app-layer merge — depending on architecture. User-visible: temporary stale reads, possible "my edit got overwritten" if conflicts. **Mitigation:** CRDT data types where applicable, pin-to-region for the few records that can't tolerate conflict.

### 14. Health check feedback loop
**Prompt:** Service has a `/health` endpoint that runs a DB query. DB is healthy but slow (2s/query during a backup window). Health checks start timing out. Walk the next 5 minutes.
**Key points:** T+0: load balancer health check timeout = 1s. DB query takes 2s. Health check fails. T+10s: LB removes the instance from rotation. Now traffic goes to fewer instances. T+30s: per-instance load doubles, more instances start failing health checks too. T+1min: cascading removal — fleet capacity collapses to 0 even though every instance is "fine." **Mitigation:** health checks shouldn't depend on slow downstreams. Use a /ready (deep) and /alive (shallow) split — LB uses shallow.

### 15. Sharded DB, double failure on one shard
**Prompt:** Sharded database, 16 shards. Shard #7's primary fails AND its replica is also down (rare double failure). 1/16th of traffic hits this shard. Walk what those users experience.
**Key points:** Users whose data lives on shard #7: 100% failure on every read or write. Users whose data is elsewhere: unaffected. From the system's perspective: 6.25% error rate — alerting may not fire if thresholds are at 10%. T+seconds: app starts seeing 5xx specifically on shard-7-bound requests; if app retries naively, retries also fail. Recovery: depends on backup restore speed, possibly hours. **Mitigation:** 3 replicas instead of 2, cross-AZ placement of replicas, fast-failover automation. *Don't* under-shard (a 16-shard cluster losing 1 shard is 6% impact; 4-shard cluster losing 1 is 25%).
