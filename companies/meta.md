# Meta calibration

## Question pool (L3–L4)

- Design Instagram's photo upload + feed display — focus on the read path for the home feed.
- Design Facebook Messenger's 1-on-1 chat — message delivery, ordering, unread state.
- Design a "people you may know" feature on a social graph (friend-of-friend traversal at scale, but bounded).
- Design notification fanout when a user posts: their N friends should see it within seconds.
- Design a comment thread on a post — pagination, ordering, real-time updates if a friend comments.
- Design a privacy-aware photo storage system: who can see what, low-latency permission checks.
- Design a typing indicator / read-receipt feature in a messaging app.

Avoid: full TAO internals (Staff-level graph cache design), full ad targeting / ML pipeline (out of scope for L3–L4).

## Rubric weights

- **weight up:**
  - `rubrics/hld.md → scoping` (Meta interviewers expect product-shaped scoping — "what does the user actually do, what are they reading vs writing?")
  - `rubrics/hld.md → data-flow` (read paths in social graphs are nontrivial)
  - `rubrics/lld.md → data-model` (graph-shaped data needs deliberate modeling, not generic relations)
- **weight down:**
  - `rubrics/lld.md → concurrency` (less central than at Google for L3–L4 prompts)

## Cultural emphasis

Meta pushes on **product sense**. The first thing they want is "do you understand who's using this and what they actually do?" Skipping the product framing and jumping into HLD is the most common L3–L4 miss. A strong answer starts with: "users post photos, friends scroll a feed, the feed needs to feel real-time but doesn't need to be perfectly fresh — we'd happily show a 30-second-old version if it's faster."

**Graph shape matters.** Most Meta problems are over a social graph: friends-of-friends, fan-out on write vs fan-in on read, "is this person allowed to see this thing." Treating these as generic relational problems with no acknowledgment of graph structure is a miss. Mentioning fan-out tradeoffs (push to friends' feeds at write time vs query each friend's posts at read time) is the right level for L3–L4.

Scale at Meta is real but secondary to product fit at L3–L4. Don't lead with "billion users" — lead with "what's the user trying to do."

## Anti-patterns (flag explicitly)

- **No product framing.** Going to HLD without articulating what the user does is the #1 Meta-specific miss.
- **Treating social-graph data as a generic relational problem.** "I'd use Postgres with a friends table" without acknowledging the read-amplification of a feed query is a flag.
- **Missing fan-out tradeoff.** Every social system has a "push at write vs query at read" decision. Not naming it is a miss.
- **Ignoring privacy / permissions on the read path.** Every Meta product has a "can this user see this?" check. Forgetting it is a hard miss for any Meta product.
- **Over-engineering for scale that's not in the prompt.** If the prompt says "Messenger 1-on-1 chat" and you're designing a 1B-user system from minute one, you've skipped the product framing.

## L3 vs L4 expectation

**L3 (entry):**
Start with product framing. Identify the headline read and write flows. Name the right primitive for the data shape. Acknowledge the social-graph fan-out tradeoff with at least one direction stated and justified. Get permission checks into the read path.

**L4 (mid):**
Same as L3 plus: explicit reasoning about feed freshness vs cost (cache TTLs, materialized views, hybrid push/pull), specific handling for celebrity/heavy-tail users (the user with 10M followers is a different problem than the user with 100), and a real-time vs near-real-time tradeoff for notifications.

## Notes

- Meta's stack (relevant context): MySQL with custom sharding, TAO (graph cache layer over MySQL), Memcached at scale, HHVM/Hack on the application tier. You don't need to name these, but reasoning about graph caching and read-heavy fanout is what's tested.
- Meta interviewers tend to be terse and conversational — answer questions directly, don't over-narrate.
