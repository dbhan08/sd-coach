# CAP theorem

**TL;DR:** In a distributed system, when the network partitions, you must choose between consistency and availability. Partition tolerance isn't optional in real systems — it's forced. So the real choice is C vs A.

## What it is

Brewer's theorem (2000). Three properties:
- **C** — Consistency. Every read returns the most recent write or an error.
- **A** — Availability. Every request gets a non-error response, eventually.
- **P** — Partition tolerance. The system continues operating despite arbitrary message loss between nodes.

You can satisfy any two but not all three. Since real networks partition (TCP timeouts, dropped packets, switch failures), P is non-negotiable. Practically, the choice is **CP** (e.g., ZooKeeper — refuses writes if quorum is lost) vs **AP** (e.g., DynamoDB with eventual consistency — keeps accepting writes, reconciles later).

## What it's for

- Framing the database/replication choice in interviews and design docs
- Explaining why a multi-region system can't simultaneously have strong consistency and five-9s availability under partition
- Setting expectations with stakeholders ("during a region failure, writes will pause for ~30 seconds — that's the C in CP")

## When to reach for it

- Choosing a database (Postgres = CP under failover; Cassandra = AP)
- Designing cross-region replication (sync = CP, async = AP)
- Explaining why "strongly consistent" and "always available" are in tension during failures

## What it's NOT good at

- **Reasoning about latency in healthy systems.** CAP only kicks in during partition. In normal operation, the relevant tradeoff is latency vs consistency. Use **PACELC**: in case of Partition, Availability vs Consistency; Else, Latency vs Consistency.
- **Single-node systems.** CAP doesn't apply — you don't have nodes to partition.
- **As a literal "pick 2" lever.** Real systems are not pure CP or pure AP — they're spectrums. Spanner is "effectively CA" by using TrueTime to bound clock skew and treating partitions as failures of the small minority of nodes.

## Alternatives + tradeoffs

| Framework | Pick when | What it adds |
|---|---|---|
| PACELC | You're reasoning about steady-state, not just failures | Adds the L (latency) tradeoff during normal operation |
| Harvest/Yield | You can serve a partial answer | Frames degraded mode as "less complete answer, still available" |
| Consistency models (linearizable / sequential / causal / eventual) | You need precise semantics, not vibes | Concrete guarantees instead of "consistency" handwave |

## Related
- [leader-election](leader-election.md)
- [cdc](cdc.md)
