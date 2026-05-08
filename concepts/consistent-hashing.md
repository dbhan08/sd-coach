# Consistent hashing

**TL;DR:** A way to distribute keys across N nodes such that adding or removing a node only moves about 1/N of the keys, instead of nearly all of them.

## What it is

Hash both your keys and your nodes onto the same circular space (e.g., 0 to 2^64). A key belongs to the next node going clockwise on the ring. When a node leaves, only its keys (the slice between it and the previous node) need to be reassigned to the next node.

In practice you use **virtual nodes** — each physical node owns 100–500 ring positions. This smooths out load (otherwise random placement gives uneven slices) and makes capacity heterogeneity easier (a "bigger" node gets more virtual positions).

## What it's for

- Distributed caches (Memcached client-side, Redis Cluster slots are a related variant)
- Sharding stateful services (each user always routes to the same node)
- DHTs (Cassandra, DynamoDB partition assignment, Chord)
- CDN edge selection ("pick the closest cache for this asset")

## When to reach for it

- Stateful nodes (you can't just throw the request anywhere — there's local state)
- Membership changes regularly (autoscaling, deploys, failures)
- You want minimal data movement on rebalance (because moving state is slow and disrupts cache hits)

## What it's NOT good at

- **Hot keys.** A single hot key still maps to one node, which gets hammered. Solutions are at a different layer: read replicas, key splitting (e.g., `user:42:counter:0` … `user:42:counter:9`), or caching the hot value at the client.
- **Range queries.** Adjacent keys in the keyspace land on random nodes — no locality. If you need to scan ranges, use a range-partitioned store (HBase-style).
- **Small N.** With <5 nodes, the load variance is high even with virtual nodes. The math gets useful around 10+ nodes.
- **Workloads where rebalance disruption is fine.** If rebalancing 100% of keys is cheap (e.g., stateless services with a shared cache layer), just use modulo hashing — it's simpler.

## Alternatives + tradeoffs

| Alternative | Pick when | Cost |
|---|---|---|
| Modulo hashing | Static cluster, stateless | Adding a node moves nearly all keys |
| Range partitioning | Need range scans, ordered keys | Hot ranges create hot shards; rebalancing is harder |
| Rendezvous (HRW) hashing | Want simpler implementation, no ring data structure | Slightly higher per-key compute (hash all nodes), comparable spread |
| Jump consistent hash | Resharding to different bucket counts (Google's algorithm) | Doesn't handle removal of arbitrary nodes — only growth/shrink at the end |

## Related
- [leader-election](leader-election.md)
