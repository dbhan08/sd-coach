# SQL vs NoSQL

**TL;DR:** "NoSQL" isn't a category — it's four different categories that share only "not relational." Pick by data shape and access pattern, not by whether SQL is uncool.

## The actual options

**Relational (SQL)** — Postgres, MySQL, SQL Server, Oracle. Data in tables with strict schema; joins; ACID transactions; SQL query language.

**Document** — MongoDB, DynamoDB (item form), Couchbase. Stores JSON-shape documents keyed by ID. Flexible schema; usually no joins; queries on document fields with secondary indexes.

**Key-value** — Redis, DynamoDB (KV form), Memcached. Just `get(key) → value`. Fastest. No queries beyond key lookup. Great for caching, sessions, counters.

**Wide-column** — Cassandra, ScyllaDB, HBase, Bigtable. Rows have a key + a sparse, dynamic set of columns. Designed for write-heavy time-series or denormalized read patterns. No joins; tunable consistency.

**Graph** — Neo4j, Amazon Neptune, ArangoDB. First-class edges; designed for "find friends-of-friends" / "shortest path" / traversal queries. Niche but right answer when traversal is the workload.

**Time-series** — InfluxDB, TimescaleDB, Prometheus. Optimized for `(time, metric, value)` ingest at high write QPS, with downsampling and retention.

**Search** — Elasticsearch, OpenSearch, Algolia. Optimized for full-text search and faceted aggregation. Used as a denormalized read view; usually fed via CDC.

## How to pick

Two-question filter:

**1. What's the query shape?**
- Joins across multiple entities → relational
- Single-key get → key-value
- Document by ID, with field-level filters → document
- High write volume on append-shape data (logs, sensor readings) → wide-column or time-series
- Full-text or fuzzy search → search engine
- Multi-hop graph traversal → graph DB

**2. What's the consistency requirement?**
- Strong, immediate, transactional → relational (or Spanner/CockroachDB at scale)
- Eventually consistent OK → most NoSQL options
- Tunable per-query → leaderless quorum (Cassandra, DynamoDB with consistent-read flag)

## Common L3–L4 traps

- **"NoSQL scales better."** Lazy. Postgres handles tens of TB and millions of reads/sec with sharding. Cassandra is great when you have *billions* of rows of a known access pattern, not because the magic word is faster.
- **"Schema-flexible is good."** Sometimes. Often it just defers schema problems to the application layer where they're harder to enforce. Schema is a feature.
- **"I'd use a graph DB for the social graph."** Sometimes — but Meta and LinkedIn use sharded relational + caching layers, not graph DBs. Graph DBs win for *deep* traversals; shallow social-graph work is fine on relational.
- **"Use Mongo because it scales."** Mongo can scale, but its pre-2018 reputation for losing writes was earned. Pick based on query shape, not vibes.

## Picking heuristic for interviews

Default to **relational**. Justify NoSQL when:
- The data shape is genuinely document, KV, wide-column, or graph
- The access pattern doesn't need joins
- The scale forces a tradeoff that relational can't handle (and you can defend the number)

## Related
- [sharding](sharding.md)
- [replication](replication.md)
- [cap-theorem](cap-theorem.md)
- [cdc](cdc.md)
