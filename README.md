# sd-coach

Personal system design interview coach. Runs as a Claude Code skill.

Four practice modes for end-to-end system design prep, calibrated for FAANG L3–L4 SWE roles. Companies seeded: Netflix, Google, Meta.

## Modes

- **mock** — full 45-min in-character interview sim with rubric-cited scorecard
- **tutor** — Socratic walkthrough, interrupts both ways, detours into Teach mid-flow
- **drill** — rapid 5-rep drills on capacity / single-component / failure-mode tracks
- **teach** — concept Q&A; builds a personal wiki at `concepts/`

## Install

Symlink this folder into your Claude Code skills directory:

```sh
ln -s "$(pwd)" ~/.claude/skills/sd-coach
```

Then `/sd-coach` is available anywhere you run `claude`.

## Use

```
/sd-coach                          # show mode menu
/sd-coach mock netflix             # full mock, Netflix calibration
/sd-coach tutor google design twitter
/sd-coach drill capacity
/sd-coach teach kafka
```

You can also paste a JD or freeform question and the skill will infer the mode.

## Status

Session 5 of 8. All four modes are live; the personal concept wiki is seeded with 5 notes. See [SESSIONS.md](SESSIONS.md) for the build plan.

| Session | What lands | Status |
|---|---|---|
| 1 | Skill skeleton, routing, README v1 | done |
| 2 | Mock mode | done |
| 3 | Tutor mode | done |
| 4 | Drill mode | done |
| 5 | Teach mode + concept wiki | done |
| 6 | Company calibration (Netflix, Google, Meta) | — |
| 7 | Rubrics + L3–L4 calibration pass | — |
| 8 | DEMO + README polish + sample sessions | — |

## How Mock works

`/sd-coach mock <company?> <topic?>` starts a 45-minute in-character interview. Seven phases: scoping → capacity → HLD → API → schema → LLD critical path → failure modes. The interviewer probes, doesn't teach, doesn't hint. If you ask "what's Kafka?" mid-mock, it gets noted for the writeup and the interview continues.

At the end you get a scorecard at `sessions/<date>-<slug>/scorecard.md` that:
- Cites specific rubric dimensions ([rubrics/hld.md](rubrics/hld.md), [rubrics/lld.md](rubrics/lld.md), [rubrics/capacity-estimation.md](rubrics/capacity-estimation.md))
- Quotes your own answers back so you can see what you actually said
- Tells you whether you cleared the L3 bar and the L4 bar, and what's missing for L4

The session folder also contains `transcript.md` (full back-and-forth) and `hld.mmd` (Mermaid of your HLD as captured during phase 3).

## How Tutor works

`/sd-coach tutor <company?> <topic?>` opens a Socratic walkthrough — no clock, no score, no interviewer character. Tutor probes with questions and **interrupts you** when you commit a structural mistake (wrong primitive, unbounded growth, missing back-pressure, wrong consistency model, etc.). You can interrupt back any time with "wait, why X?" or "pause — explain X" — that last one detours into Teach mode and returns to where you left off.

Use Tutor when you want to learn the topic, not perform under pressure. Use Mock when you want to know if you'd pass the interview today.

## How Drill works

`/sd-coach drill <track>` runs 5 rapid reps on one of three tracks: `capacity`, `component`, or `failure-modes`. Each rep is one prompt → your answer → a short model answer → score (hit/partial/miss). Difficulty starts at medium and steps up after two clean hits or down after two misses. Logs to `sessions/<date>-drill-<track>/log.md`.

Use Drill when you want reps on one specific skill — capacity math, single-component design, or failure-cascade reasoning — without the overhead of a full design walk.

## How Teach works

`/sd-coach teach <topic>` answers a concept question and writes the answer to `concepts/<slug>.md`. Asking the same concept again **updates** the existing note (preserving any user annotations marked `> note:`) rather than duplicating. Comparison questions ("Kafka vs Kinesis") write a separate comparison page and cross-link both concepts.

Teach is also auto-invoked as a detour from Mock or Tutor — say "pause — explain X" mid-Tutor, the design walk pauses, the concept gets written, and you're back where you left off.

The wiki is seeded with: [kafka](concepts/kafka.md), [consistent-hashing](concepts/consistent-hashing.md), [cap-theorem](concepts/cap-theorem.md), [cdc](concepts/cdc.md), [leader-election](concepts/leader-election.md). Add to it as you study — it's the artifact that compounds.

## How it works

- `SKILL.md` is the entry point — Claude Code loads it when `/sd-coach` is invoked.
- `SKILL.md` parses the user's input and routes to one of `modes/{mock,tutor,drill,teach}.md`.
- Each mode spec instructs Claude how to behave for that mode and references the relevant rubrics in `rubrics/` and the company calibration in `companies/<name>.md`.
- Mock and Drill write transcripts and scorecards to `sessions/` (gitignored). Teach writes/updates concept notes in `concepts/` (committed).

## Stack

Claude Code skill, markdown + Mermaid. No runtime, no dependencies, no API key.
