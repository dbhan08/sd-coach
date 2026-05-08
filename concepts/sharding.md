# Sharding

**TL;DR:** Split a dataset across N machines so each holds 1/Nth of the data. The shard key choice determines whether your queries are fast or slow.

## What it is

Horizontal partitioning of data across multiple nodes. Each node owns a *shard* — a disjoint subset of the data. A shard key (e.g., user_id) maps each record to exactly one shard.

**Four common strategies:**

1. **Hash sharding** — `shard = hash(key) % N`. Good distribution, no range queries. Re-sharding requires data movement.
2. **Range sharding** — `shard = range_lookup(key)` (e.g., users A–F on shard 0, G–M on shard 1). Range queries fast; hot ranges become hot shards.
3. **Geo sharding** — shard by user's region. Locality wins for cross-region latency, but global queries are slow. Required for some compliance regimes.
4. **By tenant / customer** — each large tenant gets its own shard (or its own DB). Strong isolation; doesn't generalize past large-tenant SaaS.

Plus **consistent hashing** as a hash variant — minimizes data movement on add/remove (see [consistent-hashing](consistent-hashing.md)).

## What it's for

- Datasets larger than a single machine's storage or RAM
- Write throughput beyond a single primary's capacity
- Fault isolation — one shard's failure affects 1/N of users, not all

## When to reach for it

- Your dataset hit single-node limits (storage, RAM, or write QPS)
- You can identify a clean shard key (one you can derive at every read/write)
- Range queries either aren't required, or you accept range-shard tradeoffs

## What it's NOT good at

- **Cross-shard transactions.** "Transfer money from user A on shard 3 to user B on shard 7" requires distributed transactions (slow, fragile) or a saga pattern (eventual consistency).
- **Cross-shard joins.** Generally, you denormalize or do app-side joins.
- **Hot keys / hot shards.** Hash sharding spreads load evenly *only if* keys are evenly used. A celebrity user with 10× normal load still pins to one shard. Mitigation: per-key sub-sharding or read replicas.
- **Choosing the wrong shard key.** Re-sharding is *expensive* — terabytes of data movement, downtime risk. Pick well.

## Alternatives + tradeoffs

| Strategy | Distribution | Range queries | Re-shard cost | Pick when |
|---|---|---|---|---|
| Hash | Even (typically) | Slow / impossible | High | Default for KV-style access |
| Consistent hash | Even | Slow | Low (move 1/N) | Frequent membership changes |
| Range | Uneven (hot ranges) | Fast | Medium | Need range scans |
| Geo | By region | Region-local fast | Medium | Cross-region latency matters |
| By tenant | Per-tenant | Per-tenant fast | None | Large-tenant SaaS |

## Related
- [consistent-hashing](consistent-hashing.md)
- [replication](replication.md)
- [leader-election](leader-election.md)
