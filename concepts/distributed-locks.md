# Distributed locks

**TL;DR:** Mutual exclusion across nodes — only one of N workers does the thing at a time. Get it wrong (clock skew, partition, lock holder crash) and you have two writers when you wanted one.

## What it is

A lock visible to multiple processes/nodes, with at-most-one-holder semantics under the assumption of an unreliable network. Implementations:

1. **etcd / ZooKeeper / Consul (CP store)** — true consensus-based locks. Holder owns an ephemeral key; key disappears if holder loses heartbeat. Strong correctness; ~10ms acquire.
2. **Redis SETNX with TTL** — atomic set-if-not-exists with expiry. Fast (~1ms) but doesn't survive Redis failover cleanly.
3. **Redlock (Redis multi-instance)** — Redis Sentinel-style lock across N Redis instances; majority wins. Faster than etcd; Martin Kleppmann famously documented correctness issues with clock skew.
4. **Database row lock** — `SELECT ... FOR UPDATE`. Simple, works if you already have the DB. Slow at scale; lock holder crash needs DB-side timeout.

## What it's for

- Distributed cron — run a scheduled job exactly once across the cluster
- Single-writer guarantee for a resource (one node updating a counter)
- Leader election (lightweight version) — first to acquire is leader
- Critical-section coordination across services

## When to reach for it

- Need strict "only one at a time" across multiple processes
- Workload doesn't shard cleanly (if it did, you wouldn't need a lock)
- The thing being locked is short-lived (≤30 seconds typical)

## What it's NOT good at

- **Long-held locks.** If you hold a lock for minutes, you're going to get bitten by network partitions and process crashes. Long-held = bad lock + good fence (see fencing tokens below).
- **Hot resources.** Every operation through a lock is a serialization bottleneck. If you can shard the resource so each shard has its own lock, do that instead.
- **Replacing transactions.** A lock isn't atomicity. If you're doing multi-step ops, you may also need transactions or 2PC.
- **Defending against malicious clients.** Locks are a coordination tool, not a security one. A misbehaving client that ignores the lock just breaks correctness.

**Fencing tokens:** the safest lock returns a monotonically-increasing token on acquire. The protected resource (e.g., a DB write) checks the token: if a newer token has already been seen, reject. This prevents the "lock holder paused for GC, lock expired, new holder acquired, original holder wakes up and writes anyway" race. Always preferable when the protected resource supports it.

## Alternatives + tradeoffs

| Mechanism | Correctness | Acquire latency | Failure mode |
|---|---|---|---|
| etcd / ZK / Consul | Strong (consensus) | ~10ms | Slow during partitions |
| Redis SETNX + TTL | OK at low risk | ~1ms | Lock holder crash leaves stale key for TTL window |
| Redlock | Disputed (clock skew) | ~3ms | Documented correctness issues at the edge |
| DB row lock | Strong (within DB) | DB latency | DB is the bottleneck |
| Sharding (no lock!) | N/A | None | None — best when applicable |
| Optimistic concurrency (CAS) | Strong | Single op | High contention = many retries |

## Related
- [leader-election](leader-election.md)
- [cap-theorem](cap-theorem.md)
- [idempotency-keys](idempotency-keys.md)
