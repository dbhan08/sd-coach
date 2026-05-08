# CDC (Change Data Capture)

**TL;DR:** A stream of row-level changes (INSERT/UPDATE/DELETE) emitted from a database in commit order, used to keep other systems eventually consistent with it without dual-writes.

## What it is

Capture changes from a source database's write-ahead log (Postgres logical replication, MySQL binlog, MongoDB oplog) and publish them as events to a downstream system. The standard implementation is **Debezium** reading the source log and writing to **Kafka**, where downstream consumers (search indexes, caches, analytics) read from.

The key idea: you don't write twice. The application writes only to the primary DB. CDC handles the propagation. Failure of the downstream system doesn't lose events because they're durably in the log/Kafka and the consumer can resume from its offset.

## What it's for

- DB → search index sync (Postgres → Elasticsearch / OpenSearch)
- Cache invalidation (DB write → invalidate the matching cache key)
- CQRS read models (DB → denormalized read view)
- Real-time analytics (DB → ClickHouse / data warehouse)
- Migrating from one DB to another with zero downtime
- Replacing brittle dual-write patterns

## When to reach for it

- You have a primary OLTP store and want to feed N derived stores
- Dual-writes are causing inconsistencies under failure
- The derived store can tolerate some lag (typical: 10ms–10s)
- You need a durable event log of all DB changes for replay/audit

## What it's NOT good at

- **Synchronous propagation.** CDC is async — there's no read-your-write across stores. If the user writes then reads from a different store, they may not see their write yet.
- **Strict cross-system transactions.** Each downstream is independently eventually consistent. If you need "either both write or neither," you need a distributed transaction protocol, not CDC.
- **Frequent schema changes.** Adding/dropping columns mid-stream needs a coordinated backfill plan, schema-registry hygiene, and consumer logic for missing fields. Otherwise consumers break silently.
- **Multi-master setups.** CDC assumes a single source of truth per row. If two masters can write to the same row, you've got conflict resolution to design — CDC alone won't help.

## Alternatives + tradeoffs

| Alternative | Pick when | Cost |
|---|---|---|
| Dual writes | Never, for the unwary | Race conditions; partial-write inconsistency under failure |
| Trigger-based publishing | Source DB doesn't expose its log | Triggers add write latency; throughput-limited |
| Outbox pattern | You can change the application | Write events to an outbox table in the same transaction; separate process publishes. Simpler than log-scraping but adds write amplification. |
| Event sourcing | Greenfield project, willing to commit | Events ARE the source of truth; no CDC needed; much bigger architectural change |
| Polling | Tiny data, low frequency | Poll lag; misses deletes unless soft-delete is used |

## Related
- [kafka](kafka.md)
