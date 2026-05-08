# Replication

**TL;DR:** Keep N copies of your data across N nodes so a single node failure doesn't lose data and reads can scale. Three topologies, two sync modes — the combination determines your tradeoffs.

## What it is

Maintaining multiple copies of the same data on different nodes. The differences:

**Topology:**
1. **Leader–follower (single-leader)** — one node accepts writes, replicates to followers. Followers serve reads. Standard for Postgres, MySQL, MongoDB, Kafka per-partition.
2. **Multi-leader** — multiple nodes accept writes, replicate to each other. Conflict resolution required. Used for cross-region active-active.
3. **Leaderless** — clients write to N replicas, read from M; quorum-based. Cassandra, DynamoDB.

**Sync mode (for leader–follower):**
1. **Synchronous** — leader waits for follower(s) to acknowledge before responding to client. No data loss on leader failure. Slower writes, vulnerable to follower slowness.
2. **Asynchronous** — leader responds immediately, replicates in background. Fast writes; possible data loss on leader failure (last few seconds).
3. **Semi-synchronous** — wait for at least one follower (any of N). Compromise.

## What it's for

- Durability — survive single-node failure without data loss
- Read scaling — followers serve reads, leader handles writes
- Latency reduction — read from a nearby replica
- Failover — promote a follower when leader dies

## When to reach for it

- Always for any production database with state you can't afford to lose
- Read-heavy workloads where read throughput exceeds single-leader capacity
- Cross-region deployments where round-trip latency to the leader is unacceptable

## What it's NOT good at

- **Improving write throughput.** Single-leader replication caps writes at the leader's capacity. For more writes, shard — replication and sharding solve different problems.
- **Strong consistency for free.** Async replication means follower reads can return stale data. Sync replication slows every write.
- **Multi-leader without conflict pain.** If the same record can be written in two regions, you've signed up for conflict resolution. CRDTs help for some data shapes; for others, manual merge.

## Alternatives + tradeoffs

| Topology + sync | Durability on leader fail | Write latency | Pick when |
|---|---|---|---|
| Leader–follower, sync | Strong (no loss) | High (wait for replica) | Compliance requires no data loss |
| Leader–follower, async | Last seconds may be lost | Low | Most production cases (Postgres default) |
| Leader–follower, semi-sync | One replica's worth safe | Medium | Want some durability without full sync cost |
| Multi-leader (multi-region) | Region-local | Low (local writes) | Cross-region active-active; tolerate conflicts |
| Leaderless quorum (R+W>N) | Configurable | Configurable | Want tunable consistency (Cassandra) |

## Related
- [cap-theorem](cap-theorem.md)
- [leader-election](leader-election.md)
- [sharding](sharding.md)
