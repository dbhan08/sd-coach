# Drill mode

**Status:** stub — full implementation in Session 4 of [SESSIONS.md](../SESSIONS.md).

## Behavior right now

When this stub is invoked, respond with:

> Drill mode isn't built yet — it lands in Session 4. The full version will run a 5-rep drill on one of three tracks: capacity estimation, single-component deep-dive, or failure-mode walk. Difficulty adjusts within the session based on how you perform.
>
> Currently working modes: none yet (we're in Session 1). Available stubs: `mock`, `tutor`, `drill`, `teach`. Run `/sd-coach` with no args to see the menu.

Then stop. Do not run any drills — this stub explicitly does not.

## What this mode will do (Session 4 spec preview)

- Three drill tracks: `capacity`, `component`, `failure-modes`
- 5 reps per session, each with prompt → user answer → model answer → score
- In-session difficulty adjustment: easier after two misses, harder after two clean hits
- L3–L4 difficulty floor and ceiling
- Writes `sessions/<date>-drill-<track>/log.md`

## What this mode will never do

- Pretend to score reps it didn't actually run
- Drift into Mock or Tutor scope (drills are short, focused, one-skill reps)
