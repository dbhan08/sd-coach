# Kafka

**TL;DR:** Distributed append-only log with partition-level ordering and consumer-group semantics. The default answer for "I need streams of events with replay."

## What it is

A topic is split into N partitions. Each partition is a totally-ordered, append-only sequence of records. Consumers track their offset (where they've read up to). Brokers replicate partitions across nodes for durability (replication factor, typically 3).

Producers pick a partition (round-robin, by key hash, or custom). Consumers in a group split partitions among themselves — each partition is read by exactly one consumer in the group at a time, which is how Kafka scales horizontally without losing ordering within a key.

## What it's for

- Event streaming between services (decouple producer from consumer)
- Log aggregation (one topic per service, central pipeline)
- CDC sinks (Debezium → Kafka → downstream)
- Replayable audit trail / event sourcing backbone
- Buffering bursty traffic before slower consumers

## When to reach for it

- Sustained throughput ≥10K msg/sec
- Multiple consumer groups need the same stream (Kafka shines vs SQS here — fan-out is free)
- You need replay (consumer crashes and re-reads from offset, or new consumer joins and reads from start)
- Ordering required within a key (use the key as partition key — same key always goes to same partition)

## What it's NOT good at

- **Low throughput.** A 3-broker cluster + ZK/KRaft + monitoring is operational overkill for <1K msg/sec workloads. Use SQS, RabbitMQ, or even just a database table.
- **Strict total ordering across the topic.** Ordering is per-partition. If you need global ordering, you're stuck with a single partition, which caps throughput.
- **Sub-millisecond latency.** Kafka end-to-end is typically 5–50ms (producer batch + replication ack + consumer poll).
- **Small messages with high fan-out per consumer.** RabbitMQ's smart routing is better suited for "dispatch this small task to one of many workers."
- **Schema-flexible message formats.** Without Schema Registry + Avro/Protobuf, schema drift becomes a debugging nightmare.

## Alternatives + tradeoffs

| Alternative | Pick when | Cost |
|---|---|---|
| Kinesis | You're on AWS and want managed; lower throughput needs (≤1MB/s/shard) | Lower max throughput per shard, harder cross-region |
| Pulsar | You need geo-replication or tiered storage out of the box | Smaller ecosystem, more moving parts (BookKeeper) |
| RabbitMQ | Task queue with smart routing, low-throughput | No replay, no log semantics, harder to scale horizontally |
| SQS / SNS | Simple decoupling, AWS-managed, no ordering/replay needed | No replay, no consumer groups, ordering only via FIFO queues with throughput limits |
| Redis Streams | Small-scale streaming inside an existing Redis deployment | Single-node bottleneck, not durable enough for critical data |

## Related
- [cdc](cdc.md)
- [leader-election](leader-election.md)
