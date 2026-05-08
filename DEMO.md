# DEMO

Reproducible command sequence for each mode. Assumes you've installed the skill (`ln -s "$(pwd)" ~/.claude/skills/sd-coach`) and started a fresh `claude` session.

## 60-second sanity check

Confirm the skill is loaded and routes:

```
/sd-coach
```

**Expected:** A four-line mode menu (Mock, Tutor, Drill, Teach) and a prompt asking which mode you want.

```
/sd-coach mock netflix
```

**Expected:** In-character interviewer kickoff like "Today's problem: [picked from Netflix's question pool]. You have 45 minutes…" Phase 1 (scoping) begins.

If neither works: the skill isn't loaded. `ls ~/.claude/skills/sd-coach` should resolve to this folder via symlink. If it doesn't, recreate the symlink and restart `claude`.

## Mock — full 45-minute interview

```
/sd-coach mock google
```

**Expected:**
1. Topic picked from `companies/google.md`'s question pool (typically URL shortener, rate limiter, web crawler, or autocomplete).
2. Phase 1: scoping. Interviewer asks open question, you reply, interviewer probes briefly, transitions to phase 2.
3. Phases continue: capacity → HLD (interviewer captures Mermaid in `hld.mmd`) → API → schema → LLD → failure modes.
4. End-of-mock: scorecard written to `sessions/<date>-<slug>/scorecard.md`, printed inline.

**Verify:**
- `ls sessions/<date>-<slug>/` shows `transcript.md`, `hld.mmd`, `scorecard.md`.
- The scorecard cites at least 2 dimensions in the form `rubrics/hld.md → <dimension>: "<your quote>" | <judgment> [L3/L4 hit|partial|miss]`.
- Interviewer never broke character to teach; deferred concept questions appear in `[deferred-questions]` of the transcript.

See [examples/mock-url-shortener/](examples/mock-url-shortener/) for an illustrative scorecard shape.

## Tutor — Socratic walkthrough

```
/sd-coach tutor design rate limiter
```

**Expected:**
1. Tutor opens with "Walk me through your design — start with what you think the system needs to do."
2. You describe; Tutor asks questions, doesn't lecture.
3. Make a deliberate structural mistake (e.g., "Redis with a 60-second TTL on the count" — fixed-window vulnerability). Tutor should interrupt with `[tutor-interrupt]`.
4. Mid-flow, ask "pause — explain X" for any concept. Tutor responds with `[detour: X]`, writes `concepts/<x>.md`, then asks "Resume?"
5. Tutor never produces a scorecard.

**Verify:**
- `ls sessions/<date>-tutor-<slug>/` shows `tutor-transcript.md`.
- Transcript contains at least one `[tutor-interrupt]` and at least one `[detour]` annotation.
- `concepts/<detour-topic>.md` exists.

See [examples/tutor-rate-limiter/](examples/tutor-rate-limiter/) for an illustrative trace.

## Drill — 5 rapid reps

```
/sd-coach drill capacity
```

**Expected:**
1. Tutor confirms track, says "5 reps, starts at medium," waits for "ready."
2. Five reps. Each is one prompt → your answer → score → 2–4 sentence model answer.
3. Difficulty steps up after 2 consecutive hits, down after 2 misses. Capped easy/hard.
4. End-of-drill tally + trend.

**Verify:**
- `ls sessions/<date>-drill-capacity/` shows `log.md`.
- The log shows 5 reps with quoted answers, scores (hit/partial/miss), and a tally section.
- Difficulty changed at least once if you had a streak.

See [examples/drill-capacity/](examples/drill-capacity/) for an illustrative log.

## Teach — concept Q&A and wiki

```
/sd-coach teach kafka
```

**Expected:**
1. Teach reads existing `concepts/kafka.md` (it's seeded).
2. Acknowledges the existing TL;DR.
3. Answers your specific framing.
4. If you ask a follow-up like "compare to Pulsar," the file gets a dated `### compare to Pulsar — <date>` subsection — does NOT overwrite the existing note.

**Verify:**
- `concepts/kafka.md` has the new subsection added, original content preserved.
- Any line starting with `> note:` is preserved verbatim.

See [concepts/](concepts/) for the seeded wiki — every note there is what Teach mode produces.

## Company calibration

Compare the same prompt under different company calibrations:

```
/sd-coach mock netflix design notification fanout
/sd-coach mock google design rate limiter
/sd-coach mock meta design feed
```

**Expected:** Different rubric weight emphasis in each scorecard:
- Netflix scorecard pushes hardest on availability, fallbacks, multi-region.
- Google scorecard pushes hardest on capacity rigor, fundamentals, tail latency.
- Meta scorecard pushes hardest on product framing, graph shape, fan-out tradeoffs.

The differences trace back to `companies/<name>.md` — open those files to see the weight overrides and cultural emphasis.

## Reset / cleanup

`sessions/` is gitignored — safe to `rm -rf` if it gets cluttered. `concepts/` is committed; only edit notes if you want changes preserved (you can also append `> note: <my thought>` lines that Teach will preserve on future updates).
