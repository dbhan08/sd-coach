# Idempotency keys

**TL;DR:** A client-supplied identifier that lets a server safely retry a request without performing the operation twice. Necessary for any operation with cost or side effects (charges, sends, deletes) over an unreliable network.

## What it is

The client generates a unique key (UUID) for an operation and passes it in a header (typically `Idempotency-Key`). The server stores `(key, request_hash, response)` for some retention period. On a request:

1. If `key` doesn't exist → process the request, store key + response, return.
2. If `key` exists with matching `request_hash` → return the cached response. Same operation, retried — no double-execute.
3. If `key` exists with different `request_hash` → reject with 409 (or 422). The client is reusing a key for a different request — almost certainly a bug.

## What it's for

- Payment APIs (Stripe pioneered the pattern publicly)
- Any non-idempotent POST that the client might retry (network error, timeout, page refresh)
- Email/SMS sends that should never be sent twice
- Order placement, ticket purchases, anything with side effects + retries

## Why it matters

Networks lose responses. Without idempotency keys:
- Client sends `POST /charge` ($100)
- Server processes, charges customer
- Server sends 200 response
- Response is lost in transit
- Client times out, retries
- Server charges customer **again**

With idempotency keys:
- Same retry now arrives with the same `Idempotency-Key`
- Server sees the key already exists → returns the cached 200, no duplicate charge

## When to reach for it

- POST/PUT operations with side effects on retry
- Any "exactly-once" requirement at the API layer
- Long-running operations where the response might be lost (e.g., workflow triggers)

## What it's NOT good at

- **GETs and idempotent DELETEs.** Already idempotent by HTTP semantics. Idempotency keys add nothing.
- **Replacing transactions.** Idempotency-key handling within the request path needs to be transactional itself — read-key, decide-action, write-key + side-effect must be atomic. Otherwise you can race two concurrent retries.
- **Forever retention.** Keys are stored until retries are unlikely (typically 24h). Beyond that, the operation should be re-attempted with a fresh key.
- **Cross-system coordination.** Idempotency at the API boundary doesn't propagate to downstream calls — you may need separate idempotency at each hop.

## Implementation notes

- Store `(key, request_hash, response, status, expires_at)` in a transactional store (Postgres, DynamoDB).
- Hash the request body so that "same key + different body" is a 409 (catches client bugs).
- Lock the key while processing (`SELECT ... FOR UPDATE` or equivalent) so concurrent retries don't both execute.
- Include the response status code in the cache so retries get exactly the original response.
- TTL of 24 hours is industry standard (Stripe).

## Alternatives + tradeoffs

| Approach | Pick when | Cost |
|---|---|---|
| Idempotency keys | API boundary, retryable ops | Storage + lock overhead per request |
| Make ops naturally idempotent (CAS, version numbers) | Internal operations | Requires schema design — versioned writes |
| At-least-once delivery + dedup downstream | Async event-driven | Dedup logic everywhere downstream |
| Two-phase commit | Multi-system transactions | Slow, fragile, rarely worth it |

## Related
- [rate-limiting](rate-limiting.md)
- [distributed-locks](distributed-locks.md)
- [cdc](cdc.md)
