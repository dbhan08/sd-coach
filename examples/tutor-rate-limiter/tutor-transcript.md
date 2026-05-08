# Tutor — Rate limiter — synthetic example

**Synthetic — illustrative only.** Not a real run.

Company emphasis: none
Start: 2026-05-07T15:00:00

---

## Walk

**Tutor:** Walk me through your design — start with what you think the system needs to do.

**User:** I want to limit each user to 100 requests per minute on a public API. If they go over, return 429.

**Tutor:** Good start. Where does the count live?

**User:** In Redis. Each user has a key with their request count, and a TTL of 60 seconds.

[tutor-interrupt] You said "TTL of 60 seconds" on the count. Walk me through what happens to a user who sends a burst of 200 requests at second 59 of one window.

**User:** Hmm. They'd hit 100 in the first window and get rate-limited for the last second of that window. But then at second 0 of the next window, the TTL expires and they have a fresh 100. So they could send 100 + 100 = 200 requests in a 1-second span across the window boundary.

**Tutor:** Right. That's the classic fixed-window problem — bursts at the boundary. What would you do about it?

**User:** Sliding window? But how do I implement that — store every request timestamp?

[user-interrupt] Wait, "pause — explain sliding window log vs sliding window counter."

[detour: sliding-window-rate-limiting] — see concepts/sliding-window-rate-limiting.md

**Tutor:** Detour: sliding-window-rate-limiting. I'll write this to concepts/sliding-window-rate-limiting.md.

> **Sliding window log:** store every request timestamp per user. To check the rate, count timestamps within the last N seconds, evict older ones. Exact, but memory grows with throughput per user — at high QPS this is expensive.
>
> **Sliding window counter:** track counts per fixed sub-window (e.g., per second), then sum the most recent N. Approximate (you lose intra-second resolution) but bounded memory. The standard production choice.
>
> **Token bucket:** orthogonal pattern. Tokens refill at a fixed rate; each request consumes a token; if zero, deny. Naturally handles bursts up to the bucket size.

Resume tutor?

**User:** Yes — let's go with token bucket.

**Tutor:** OK, back to where we paused — you're picking token bucket over sliding window. Why?

**User:** Two reasons. One, it's simpler — single integer per user (current tokens) plus a refill timestamp. Two, it allows controlled bursts: if a user has 100 tokens and refills at 100/min, they can spend all 100 at once and then wait. That maps to real user behavior better than a hard 100/min cap.

**Tutor:** Where do you store the bucket state?

**User:** Redis again. Hash per user with `tokens` and `last_refill_ts`. Atomic update via Lua script — read both, compute new tokens based on time elapsed, decrement, write back.

[tutor-interrupt] You said "atomic via Lua." Walk me through what happens if Redis is down for 30 seconds.

**User:** Right, that's a single point of failure. Two options. One: replicated Redis with failover — simple but lose some writes. Two: fail-open at the rate limiter — if Redis is down, allow requests rather than block. That's a tradeoff between availability and accurate rate limiting.

**Tutor:** Which one would you pick for this system?

**User:** Fail-open. The cost of false-blocking real users during a Redis outage is worse than letting through some over-quota requests for 30 seconds. Plus, the upstream service we're protecting probably isn't going to be hit hard enough by a 30-second window of unrate-limited traffic to matter.

**Tutor:** That's a defensible answer. I'd push back on one thing: that argument depends on what's downstream. If your downstream is a payment processor, "let everything through" is much worse than for a search API. The right answer is "fail-open *for read APIs*, fail-closed *for write APIs that have real cost per request*." Note that for the writeup.

**User:** Got it.

**Tutor:** Now, what about cross-region? You have one Redis cluster in us-east-1, but users hit your service from us-west-2.

[... walk continues ...]

---

## End: 2026-05-07T15:35:00

## Concepts written this session
- sliding-window-rate-limiting (new file)
