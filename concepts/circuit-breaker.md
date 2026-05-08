# Circuit breaker

**TL;DR:** A wrapper around outbound calls that "trips" (stops calling) when a downstream is unhealthy, then probes to see if it's recovered. Prevents one slow downstream from cascading into total outage.

## What it is

A state machine with three states:

- **Closed** — calls pass through normally. Track failure rate (e.g., last 100 requests, last 60 seconds).
- **Open** — calls fail fast (return error immediately, don't even attempt the downstream). Saves the calling thread / connection from blocking on the slow downstream.
- **Half-open** — after a cooldown (typically 30s), allow a few probe calls. If they succeed, transition back to closed. If they fail, return to open.

Trip threshold: typically "≥50% errors in the last 100 calls" or similar. Tunable per-call-site.

## What it's for

- Protecting your service from a slow / failing downstream
- Failing fast so callers see "downstream unavailable" in 1ms instead of "request timeout" in 30s
- Preventing retry storms — if downstream is down, immediate failure means client backs off rather than piling on
- Buying time for the downstream to recover (no traffic = no load while it heals)

## When to reach for it

- Sync calls to any downstream that *can* fail (which is all of them)
- Microservice architectures where one service's slowness shouldn't kill the caller
- Calls to external services with their own SLO (third-party APIs)

## What it's NOT good at

- **Saving you when retries are the problem.** Circuit breaker is a *complement* to retry-with-backoff, not a replacement. Without backoff, you'll just retry into the open breaker and burn CPU.
- **Granular protection.** A single breaker covers a whole call-site / dependency. If 50% of calls to a downstream fail because of a specific endpoint, a coarse breaker might trip for the healthy 50% too. Some implementations break per-endpoint.
- **Slow vs failed distinction.** A breaker that trips on errors won't trip on a downstream that's slow but technically responding. Breakers based on latency thresholds catch this — Hystrix-style circuit breakers historically combined both.
- **Cascading failure prevention by itself.** Need it + bounded retries + bulkheads (separate connection pools per downstream) + good timeouts. Circuit breaker is one tool in the resilience kit.

## Half-open probe behavior

The half-open transition matters. Two common strategies:
1. **Single probe** — let one request through. Pass = closed; fail = open. Simple.
2. **Gradual recovery** — let increasing percentage of traffic through over a window. Smoother for high-QPS callers.

If you spam probes in half-open and downstream is just barely recovering, you can re-trip immediately. Probe rate matters.

## Alternatives + tradeoffs

| Pattern | Protects against | Pick when |
|---|---|---|
| Circuit breaker | Failing downstream | Default for any non-trivial outbound call |
| Bulkhead (separate pools) | One downstream exhausting all connections | Multiple critical downstreams |
| Retry with exp backoff + jitter | Transient blips | Combined with breaker |
| Timeout | Anything slow | Always — pair with breaker |
| Bounded queue / backpressure | Producer outpacing consumer | Async / streaming pipelines |
| Rate limiter | Caller-side | Smoothing self-imposed load |

## Implementation note

Resilience4j (Java), Polly (.NET), gobreaker (Go), opossum (JS) — many language-native libraries. Don't roll your own; subtle bugs in state transitions and metric windows.

## Related
- [rate-limiting](rate-limiting.md)
- [load-balancer](load-balancer.md)
- [idempotency-keys](idempotency-keys.md)
