# Load balancer

**TL;DR:** Distributes incoming traffic across N backend servers. Two layers (L4 / L7), several algorithms — pick based on what you need to inspect and how sticky sessions need to be.

## What it is

A node (or service) that receives client requests and forwards them to one of several backend servers. Two operational layers:

**L4 (transport layer)** — operates on TCP/UDP packets. Doesn't read the request body; just routes by IP + port. Fast, simple, can't make app-aware decisions.
- Examples: AWS NLB, HAProxy in TCP mode, IPVS

**L7 (application layer)** — terminates the connection, parses HTTP. Can route by path, header, method, body. Slower per-request, much more flexible.
- Examples: AWS ALB, NGINX, Envoy, HAProxy in HTTP mode

**Algorithms:**
- **Round-robin** — backend N+1 each request. Simple, ignores backend load.
- **Least-connections** — pick the backend with fewest open connections. Better for variable-duration requests.
- **Weighted (RR or least-conn)** — give bigger backends more weight. Useful for heterogeneous fleets.
- **Hash-based (consistent hash)** — route by client IP, session ID, or URL — same client always lands on same backend. Used for cache locality or sticky sessions.
- **Response-time-aware (least-time)** — pick the backend with lowest recent p99. Sounds great, hard to tune.

## What it's for

- Spreading load across a fleet so no one server is the bottleneck
- Failure isolation — remove an unhealthy backend from rotation
- TLS termination (L7) — terminate encryption once, talk plain HTTP to backends
- Path-based routing — `/api/*` to API service, `/static/*` to CDN

## When to reach for it

- More than one backend instance (else what are you balancing?)
- Health-check / failover requirements
- TLS termination + HTTP routing in one place
- Geographic / weighted routing for region-based traffic split

## What it's NOT good at

- **Replacing a CDN.** LB is for your origin; CDN serves cached static content from edge. Different layer.
- **Solving database load.** LBs distribute requests, not data. For DB, you need replication or sharding.
- **Magic.** LBs route — they don't know if a backend is *correct*, only if it's *responsive*. Health checks must be designed thoughtfully (a /health endpoint that hits the DB will cascade-fail when DB is slow).
- **Magic again.** Hash-based stickiness gives session locality but creates hot backends if traffic is skewed. Doesn't actually help much with cache locality if backends share a cache layer.

## Alternatives + tradeoffs

| Choice | Inspect request | Latency overhead | Pick when |
|---|---|---|---|
| L4 (TCP) | No (just IP/port) | Microseconds | Highest throughput, no app-level routing needed |
| L7 (HTTP) | Yes (path, header, body) | Milliseconds | Path routing, TLS termination, app-aware logic |
| DNS round-robin | No (resolution time) | DNS-cache-bound | Cheap and simple; bad failover |
| Anycast | No (BGP routing) | Network-level | Geo-distributed, single-IP frontend |

**Algorithm choice:**
- Default: round-robin or least-connections
- Long-lived connections (websockets): least-connections
- Cache locality matters: consistent hash by user
- Heterogeneous backend sizes: weighted

## Related
- [consistent-hashing](consistent-hashing.md)
- [circuit-breaker](circuit-breaker.md)
