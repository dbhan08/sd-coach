# Caching strategies

**TL;DR:** A cache is two decisions: how do writes interact with the cache (write strategy), and how do reads (read strategy + eviction). Different combinations have different consistency and complexity properties.

## What it is

A cache holds a subset of data in fast storage (RAM, SSD, edge) for fast access. The strategies define the protocol between the cache, the application, and the underlying authoritative store.

**Read strategies:**
- **Cache-aside (lazy load)** — app checks cache; on miss, reads from DB and populates cache. Most common. App owns invalidation.
- **Read-through** — cache itself fetches from DB on miss. App talks only to cache. Cleaner abstraction; cache layer is more complex.
- **Refresh-ahead** — cache proactively reloads entries before TTL expires. Good for predictable hot keys; wasted if entry isn't read.

**Write strategies:**
- **Write-through** — app writes to cache, cache writes synchronously to DB. Cache always consistent with DB; write latency = DB write.
- **Write-back (write-behind)** — app writes to cache; cache flushes to DB asynchronously. Fast writes; risk of data loss if cache fails before flush.
- **Write-around** — app writes directly to DB; cache only loaded on read. Avoids polluting cache with rarely-read writes.

**Eviction policies:**
- **TTL** — entries expire after a fixed time
- **LRU** — evict least-recently-used when full
- **LFU** — evict least-frequently-used (better for skewed distributions)
- **FIFO** — evict oldest (simple, often worse than LRU)

## What it's for

- Reducing read latency (DB at 50ms → memory at 1ms)
- Reducing DB load (1000 cache hits per miss = 1/1000th the DB load)
- Reducing cost (DB ops are usually 100× more expensive than cache hits)

## When to reach for it

- Read-heavy workloads with skewed key distributions (Pareto: 80% of reads on 20% of keys)
- Slow downstream you can't speed up directly
- Computed-and-cacheable values (rendered HTML, search results, expensive joins)

## What it's NOT good at

- **Write-heavy workloads.** Cache invalidation cost can exceed cache benefit.
- **Strong consistency under writes.** If your cache and DB can diverge, your reads might see stale data — for many use cases acceptable, but say so explicitly.
- **The thundering-herd / cache-stampede.** When a hot key expires, 1000 concurrent reads all miss and hit the DB at once. Mitigations: request coalescing (singleflight), staggered expiry, lock-and-refill.

## Alternatives + tradeoffs

| Strategy combo | Consistency | Write latency | Failure mode |
|---|---|---|---|
| Cache-aside + write-around | Eventually consistent | DB write only | Stale reads briefly |
| Read-through + write-through | Strongly consistent | Cache + DB write | Slow writes |
| Read-through + write-back | Eventually consistent (best perf) | Cache write only | Data loss on cache crash |
| Refresh-ahead + write-around | Eventually consistent | DB write only | Wasted refreshes on cold keys |

## Related
- [consistent-hashing](consistent-hashing.md)
- [cdn-basics](cdn-basics.md) *(if seeded)*
- [cdc](cdc.md)
