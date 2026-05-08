# Tutor mode

**Status:** stub — full implementation in Session 3 of [SESSIONS.md](../SESSIONS.md).

## Behavior right now

When this stub is invoked, respond with:

> Tutor mode isn't built yet — it lands in Session 3. The full version will run a Socratic walkthrough of a system design problem with bidirectional interrupts (Tutor interrupts when you make a structural mistake; you can interrupt to ask anything mid-flow), with detours into Teach mode that pause the design walk and resume.
>
> Currently working modes: none yet (we're in Session 1). Available stubs: `mock`, `tutor`, `drill`, `teach`. Run `/sd-coach` with no args to see the menu.

Then stop. Do not attempt to run a tutor session — this stub explicitly does not.

## What this mode will do (Session 3 spec preview)

- Socratic walkthrough of one design problem
- Tutor-side interrupts when the user makes a structural mistake (concrete trigger list to be defined)
- User-side interrupts: "wait, why X?", "pause — explain Y", "skip this phase"
- Detour-into-Teach hook: pauses the design walk, runs Teach, resumes
- Writes `sessions/<date>-<slug>/tutor-transcript.md` with `[interrupt]` and `[detour]` annotations

## What this mode will never do

- Score the user (no scorecard — use Mock for that)
- Run a clock or pretend there's interview pressure
- Stay silent through structural mistakes
