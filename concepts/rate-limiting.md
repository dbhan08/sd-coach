# Rate limiting

**TL;DR:** Cap how many requests a client can make per unit time. Five common algorithms; the choice affects burst tolerance and storage cost.

## What it is

A control that rejects (or delays) requests above a quota. Quotas are usually per-user / per-IP / per-API-key / per-endpoint. The algorithm decides what counts as "above quota" — the choice has real consequences.

**Five algorithms:**

1. **Fixed window** — count requests within `[now - 60s, now)`, reset on the minute. Simple. Suffers from boundary bursts (200 requests in the last second of one window + first second of next window).
2. **Sliding window log** — store a timestamp per request; count timestamps within the last N seconds. Exact, but storage = throughput × window. Expensive at high QPS per user.
3. **Sliding window counter** — track per-second sub-windows, sum the most recent N. Approximate, bounded memory. The standard production choice for "100 req/min" type limits.
4. **Token bucket** — N tokens, refill at rate R per second; each request consumes a token. Naturally allows controlled bursts up to bucket size.
5. **Leaky bucket** — fixed-rate queue; requests fill the bucket, drain at rate R. Smooths bursts (no spikes through the limiter). Different use case from token bucket.

## What it's for

- API gateways protecting backend services
- Public APIs preventing abuse
- Per-user quotas on expensive operations (search, image upload)
- Smoothing bursty internal traffic (leaky bucket)
- Cost control on metered downstream services

## When to reach for it

- Public-facing endpoint with no caller-imposed concurrency control
- Downstream service has hard QPS limits or per-request cost
- You need to prevent one tenant from starving others (multi-tenancy)
- Anti-abuse / DDoS mitigation at the application layer

## What it's NOT good at

- **Dropping huge volumetric DDoS at the application layer.** That's a CDN/WAF's job. App-level rate limiters can be overwhelmed by the request volume itself.
- **Fairness across heterogeneous users.** A flat 100/min hurts heavy users; weighted/tier-based limits add complexity.
- **Handling distributed state cheaply.** A globally-coordinated rate limiter requires Redis or similar; per-region limiters drift but are cheaper.

## Alternatives + tradeoffs

| Algorithm | Burst tolerance | Storage / user | Pick when |
|---|---|---|---|
| Fixed window | Bad (boundary bursts) | 1 counter | Throwaway prototype only |
| Sliding window log | Exact | O(throughput × window) | Tiny user count, exact correctness needed |
| Sliding window counter | Good (smoothed) | O(window/sub-window) | Default production choice |
| Token bucket | Configurable burst | 2 ints (tokens, last-refill) | Want to allow bursts up to N |
| Leaky bucket | None (smoothed out) | Queue depth | Smoothing, not just rejecting |

## Related
- [consistent-hashing](consistent-hashing.md)
- [circuit-breaker](circuit-breaker.md)
- [idempotency-keys](idempotency-keys.md)
