# Mock — URL shortener — synthetic example

**Synthetic — illustrative only.** Not a real run.

Company: none
Target: L3–L4
Start: 2026-05-07T14:00:00

---

## Phase 1: Scoping

**Interviewer:** Today's problem: design a URL shortener. You have 45 minutes. I'll be tracking seven phases. Ask me anything before you start.

**Candidate:** OK, so it's like Bitly — user gives a long URL, we give back a short one, and clicking the short one redirects to the long one. A few questions: what's the scale we're targeting? Is this a public service or internal? Do users need accounts? And how long do short URLs persist?

**Interviewer:** Public consumer service. Assume 100M URLs created per month, 10× that in clicks. No accounts for the MVP — anyone can shorten anything. URLs persist for at least 5 years.

**Candidate:** Got it. So the headline flow is: POST a long URL, get a short code; GET the short code, get a 301 redirect. MVP doesn't have accounts, custom slugs, analytics, or expiry control. Out of scope: spam protection, brand safety. Does that match?

**Interviewer:** Yes. Move on.

[note: candidate restated scope cleanly, named what's in/out, asked about scale + persistence — `rubrics/hld.md → scoping`: L3 hit, L4 partial (didn't name an explicit NFR like latency/availability target).]

## Phase 2: Capacity estimation

**Interviewer:** Give me back-of-envelope numbers.

**Candidate:** Let me state assumptions. 100M new URLs/month — call that 100M / 30 / 86400 ≈ 40 writes/sec average. Reads are 10× writes, so ~400 reads/sec average. With a 3× peak factor, peak read QPS is ~1.2K. Storage: each record is the long URL (assume average 100 bytes) plus the short code (~7 chars) plus some metadata (~20 bytes). Call it ~200 bytes per record. 100M/month × 12 × 5 years = 6B records × 200 bytes = 1.2 TB. That fits on a single beefy node — sanity check, that's reasonable for a URL shortener.

**Interviewer:** OK, good. Onto HLD.

[note: stated assumptions, math right within order of magnitude, applied peak factor, sanity-checked storage. `rubrics/capacity-estimation.md → assumptions`: L3 hit, L4 partial (didn't identify which assumption is load-bearing). `→ math`: L4 hit. `→ sanity-checks`: L3 hit. `→ relevance`: storage number actually drove the design choice (single node = no sharding needed for MVP) — L4 hit.]

## Phase 3: HLD

**Interviewer:** Walk me through it.

**Candidate:** [draws hld.mmd] Three components for the MVP. First, an API server fronted by a load balancer — POST /shorten and GET /:code. Second, a Postgres DB storing (short_code, long_url, created_at). Third, a CDN or HTTP cache in front of GETs because reads are 10× writes and most clicks come from a small head of viral links.

**Interviewer:** Why Postgres over a KV store?

**Candidate:** For 1.2 TB and 1.2K peak read QPS, Postgres handles it easily, and the access pattern is exact-key lookup which any DB supports. The reason I'd lean Postgres: I want a unique constraint on the short_code so I can reject collisions on insert. A KV store usually doesn't enforce that as cleanly. Tradeoff: at much higher scale I'd switch to a KV store and handle collision in the app layer with retries.

**Interviewer:** What's *in* the cache, and what invalidates it?

**Candidate:** The cache holds (short_code → long_url) mappings. URLs in this system are immutable — you never re-shorten an existing code — so we don't need invalidation. TTLs only exist for cost control on cold entries. If we ever supported user-deletable URLs, that's where invalidation gets real.

**Interviewer:** Generation strategy for short_code?

**Candidate:** Two options. One: hash the long URL, take the first N base62 chars. Problem: same URL gets the same code, which is bad for analytics if we ever add them, and collisions need handling. Two: random 7-char base62, check uniqueness on insert, retry on collision. 62^7 ≈ 3.5T, way more than our 6B target, so collision rate is tiny. I'd go with option 2 — simpler, no surprises.

**Interviewer:** OK. API contract?

[note: clean component split (3 components, each with single responsibility) — `rubrics/hld.md → component-breakdown`: L4 hit. Walked the read path AND the write path, named the cache role, named what's NOT cached (write path) — `→ data-flow`: L4 hit. Justified Postgres over KV with the constraint argument AND named when they'd switch — `→ tradeoff-articulation`: L4 hit. Cache TTL discussion was good but didn't quantify the head/tail distribution that motivates it — `→ scaling`: L3 hit, L4 partial.]

## Phase 4: API contract

**Candidate:** POST /shorten with body `{long_url: string}`. Response: 201 Created with `{short_code: string, short_url: string}`. Errors: 400 if URL malformed, 422 if URL is on a known-bad list (out of scope here), 5xx for server errors. Should be idempotent — same long URL submitted twice should... hmm, by my generation strategy (random codes, not hash) it'll get a new code. That's the design. Could add idempotency via an Idempotency-Key header if a client wants stable behavior.

GET /:code returns 301 to the long URL, 404 if not found.

**Interviewer:** Onto the schema.

[note: typed inputs/outputs, error cases explicit, recognized idempotency tension and named the mitigation — `rubrics/lld.md → interface-design`: L4 hit. Pagination wasn't relevant here — n/a.]

---

*[remaining phases (schema, LLD, failure modes) elided for brevity in this synthetic example]*

---

## Deferred questions
- (none in this run)

## End: 2026-05-07T14:43:00
