# Mock mode

**Status:** stub — full implementation in Session 2 of [SESSIONS.md](../SESSIONS.md).

## Behavior right now

When this stub is invoked, respond with:

> Mock mode isn't built yet — it lands in Session 2. The full version will run a 45-minute in-character FAANG interviewer sim with phase tracker (scoping → capacity → HLD → API → schema → LLD → failure modes), no hints, and a rubric-cited scorecard written to `sessions/`.
>
> Currently working modes: none yet (we're in Session 1). Available stubs: `mock`, `tutor`, `drill`, `teach`. Run `/sd-coach` with no args to see the menu.

Then stop. Do not attempt to simulate any part of a real mock session — this stub explicitly does not run a mock.

## What this mode will do (Session 2 spec preview)

- 45-minute simulated interview, in-character as a FAANG senior engineer
- Phase tracker with explicit time budgets per phase
- No hints, no teaching, no scope renegotiation mid-session
- Writes `sessions/<date>-<slug>/transcript.md`, `hld.mmd`, `scorecard.md`
- Scorecard cites rubric dimensions verbatim and quotes the candidate's own answers back

## What this mode will never do

- Break character to teach a concept (use Tutor or Teach mode instead)
- Give hints when the candidate gets stuck
- Score generously
