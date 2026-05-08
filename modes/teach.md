# Teach mode

**Status:** stub — full implementation in Session 5 of [SESSIONS.md](../SESSIONS.md).

## Behavior right now

When this stub is invoked, respond with:

> Teach mode isn't built yet — it lands in Session 5. The full version will answer concept questions ("explain Kafka", "compare Kinesis vs SQS", "when would I use a CDC pipeline") and write each answer to a personal wiki at `concepts/<slug>.md`. Asking the same concept again updates the existing note instead of duplicating. Cross-links to related concepts.
>
> Currently working modes: none yet (we're in Session 1). Available stubs: `mock`, `tutor`, `drill`, `teach`. Run `/sd-coach` with no args to see the menu.

Then stop. Do not attempt to teach a concept or write to `concepts/` — this stub explicitly does not.

## What this mode will do (Session 5 spec preview)

- Standalone concept Q&A; also invocable mid-Mock/Tutor as a detour
- Reads `concepts/<slug>.md` before answering — updates rather than overwrites
- Concept note template: what it is / what it's for / when to reach for it / what it's NOT good at / alternatives + tradeoffs / related links
- Cross-links: each note ends with "Related: " linking to other notes in the wiki
- Seeds 5 notes during Session 5: Kafka, consistent hashing, CAP theorem, CDC, leader election

## What this mode will never do

- Overwrite an existing concept note without preserving the user's annotations
- Invent a concept page with no real content (better to say "I'd need to look this up first")
- Write generic cookie-cutter notes — each note must be tuned by the user's follow-up questions
