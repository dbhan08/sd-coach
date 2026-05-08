# Leader election

**TL;DR:** Pick exactly one node from a group of N to act as the coordinator, with mutual exclusion guaranteed even under failures and network partitions.

## What it is

A distributed consensus problem. Algorithms (rough capability ladder):
- **Bully algorithm** — simple, depends on stable IDs; works for small clusters in trusted networks
- **Paxos** — the original; correct but notoriously hard to implement and reason about
- **Raft** — the modern choice; designed for understandability. Used by etcd, Consul, MongoDB, TiKV, CockroachDB
- **ZooKeeper-style ephemeral nodes** — clients race to create an ephemeral znode; whoever wins is leader; node disappears when client disconnects. Built on top of ZAB (a Paxos variant).

In practice for a new system: don't roll your own. Use etcd / Consul / ZooKeeper as a coordination primitive.

## What it's for

- Single-writer guarantees in primary-replica DBs (Postgres failover, MongoDB primary election, Kafka controller)
- Distributed locks (only one of N workers should run this cleanup job)
- Primary failover (when the leader dies, elect the next one)
- Distributed cron (run a scheduled job exactly once across the cluster)

## When to reach for it

- You have a workload where exactly one node should be doing X at a time
- You need this guarantee under partition and node failures
- The "X" can't be cheaply sharded so each shard has its own leader (if it can, prefer that)

## What it's NOT good at

- **Hot-path coordination.** Every write going through the leader is a single-machine bottleneck. If you're hitting leader CPU/network limits, you've outgrown the pattern — shard, or move to multi-leader.
- **Avoiding coordination entirely.** The best leader election is no leader election. If you can design so each shard is independent (each user's data on one node), no global leader is needed.
- **Sub-second failover.** Election + heartbeats add seconds of unavailability under failure. For sub-second, design for active-active or use stateless leader (any node can take over instantly because state is in shared storage).
- **Frequent membership changes.** Cluster reconfiguration is a coordinated operation; flapping nodes can cause repeated elections (election storms).

## Alternatives + tradeoffs

| Alternative | Pick when | Cost |
|---|---|---|
| Multi-leader (active-active) | You can't tolerate failover latency, conflicts are rare or resolvable | Conflict resolution becomes the hard problem |
| Sharding (per-shard leaders) | Workload partitions cleanly | More moving parts, but no single bottleneck |
| Lease-based locks (Redis Redlock, ZK ephemeral) | Short-term mutual exclusion, willing to accept weaker guarantees | Not as strong as consensus — clock-skew bugs (Redlock has well-documented issues) |
| Static designation | Tiny system, manual ops, no failover | Single point of failure; ops burden |

## Related
- [consistent-hashing](consistent-hashing.md)
- [cap-theorem](cap-theorem.md)
- [kafka](kafka.md)
