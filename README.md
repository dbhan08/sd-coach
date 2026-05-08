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

Session 2 of 8. Mock mode is live; the other three are still stubs. See [SESSIONS.md](SESSIONS.md) for the build plan.

| Session | What lands | Status |
|---|---|---|
| 1 | Skill skeleton, routing, README v1 | done |
| 2 | Mock mode | done |
| 3 | Tutor mode | — |
| 4 | Drill mode | — |
| 5 | Teach mode + concept wiki | — |
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

## How it works

- `SKILL.md` is the entry point — Claude Code loads it when `/sd-coach` is invoked.
- `SKILL.md` parses the user's input and routes to one of `modes/{mock,tutor,drill,teach}.md`.
- Each mode spec instructs Claude how to behave for that mode and references the relevant rubrics in `rubrics/` and the company calibration in `companies/<name>.md`.
- Mock and Drill write transcripts and scorecards to `sessions/` (gitignored). Teach writes/updates concept notes in `concepts/` (committed).

## Stack

Claude Code skill, markdown + Mermaid. No runtime, no dependencies, no API key.
