# sd-coach

Personal system design interview coach. Runs as a Claude Code skill.

Five practice modes for end-to-end system design prep, calibrated for FAANG L3–L4 SWE roles. Works with any company (Netflix, Google, Meta, Nuro seeded; others auto-calibrate from JD or fall back to FAANG-generic).

## Modes

- **coach** — teacher walks you through a scenario, with model answers at each phase and proactive concept teaching. **Default for learners.**
- **mock** — full 45-min in-character interview sim with rubric-cited scorecard. **No teaching** — pressure-test only.
- **tutor** — Socratic walkthrough, you drive, tutor interrupts on structural mistakes and offers detours on knowledge gaps.
- **drill** — rapid 5-rep drills on capacity / single-component / failure-mode tracks.
- **teach** — concept Q&A; builds a personal wiki at `concepts/`.

**Quick decision guide:** Use **coach** if you don't know the topic well yet — it teaches you. Use **mock** when you do know it and want to pressure-test. Use **tutor** for a Socratic middle ground. Use **drill** for fast reps. Use **teach** for one-off concept questions.

## Install

Symlink this folder into your Claude Code skills directory:

```sh
ln -s "$(pwd)" ~/.claude/skills/sd-coach
```

Then `/sd-coach` is available anywhere you run `claude`.

## Use

```
/sd-coach                          # show mode menu
/sd-coach coach netflix            # teacher walks you through a Netflix scenario
/sd-coach mock google              # full mock interview, Google calibration
/sd-coach tutor meta design feed   # Socratic walk through feed design
/sd-coach drill capacity           # 5-rep capacity estimation drill
/sd-coach teach kafka              # explain Kafka, write to concepts/
```

You can also **paste a JD, role description, or freeform question** and the skill will infer the mode (defaults to coach for learners). Companies that aren't seeded (Amazon, Apple, Stripe, etc.) auto-calibrate from cues in the paste; offer to save the calibration after the session.

## Status

All 8 build sessions complete. Ready to use. See [DEMO.md](DEMO.md) for reproducible commands and [examples/](examples/) for what each mode's output looks like. Build log in [SESSIONS.md](SESSIONS.md).

| Session | What lands | Status |
|---|---|---|
| 1 | Skill skeleton, routing, README v1 | done |
| 2 | Mock mode | done |
| 3 | Tutor mode | done |
| 4 | Drill mode | done |
| 5 | Teach mode + concept wiki | done |
| 6 | Company calibration (Netflix, Google, Meta) | done |
| 7 | Rubrics + L3–L4 calibration pass | done |
| 8 | DEMO + README polish + examples | done |

## How Coach works

`/sd-coach coach <company?> <topic?>` — or paste a JD/role description — opens a teacher-led walkthrough of a scenario. The teacher:
- **Picks a scenario** appropriate for the role and company (or uses the topic you named)
- For each of the 7 phases (scoping → capacity → HLD → API → schema → LLD → failure modes):
  1. **Suggests** what you should be doing in this phase
  2. **Points out what to ask** the interviewer
  3. **Shows the model answer** at L3, with a "to clear L4" supplement
  4. **Lets you attempt** your own answer
  5. **Reconciles** — quotes you back, names what's strong/missing, cites rubric dimensions
  6. **Proactively teaches** any concept you used shallowly (small things inline; bigger ones detour into Teach and write `concepts/<topic>.md`)
- **Builds a `model-answer.md`** as you go, so you leave with a complete reference for that scenario

Coach is the right mode when you don't know the topic well yet. It hand-holds; that's the point. Once you've done a few Coach walks on a topic, switch to **mock** for the same scenario to pressure-test without help.

For unknown companies (anything not in `companies/`), Coach reads the role/JD/paste for cues and synthesizes a one-time calibration. After the session it offers to save that as `companies/<name>.md` for next time.

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

`/sd-coach drill <track>` runs 5 rapid reps on one of four tracks: `capacity`, `component`, `failure-modes`, or `concepts`. Each rep is one prompt → your answer → a short model answer → score (hit/partial/miss). Difficulty starts at medium and steps up after two clean hits or down after two misses. Logs to `sessions/<date>-drill-<track>/log.md`.

- `drill capacity` — 15-prompt bank at [questions/drill-capacity.md](questions/drill-capacity.md). Estimation reps from B2B SaaS sizing through global egress bills.
- `drill component` — 15-prompt bank at [questions/drill-component.md](questions/drill-component.md). One-component-at-a-time reps from session stores through distributed counters.
- `drill failure-modes` — 15-prompt bank at [questions/drill-failure-modes.md](questions/drill-failure-modes.md). System sketch + failure event → walk the cascade.
- `drill concepts <name?>` — quiz reps drawn from your `concepts/` wiki. `drill concepts kafka` targets one concept; `drill concepts` randomizes across all your notes. Active recall, not just rereading.

Use Drill when you want reps on one specific skill without the overhead of a full design walk.

## How Teach works

`/sd-coach teach <topic>` answers a concept question and writes the answer to `concepts/<slug>.md`. Asking the same concept again **updates** the existing note (preserving any user annotations marked `> note:`) rather than duplicating. Comparison questions ("Kafka vs Kinesis") write a separate comparison page and cross-link both concepts.

After Teach explains a concept, it offers a quick **3–5 question quiz** to check understanding. Quiz mixes recall, application, tradeoff, and failure-mode questions. Tally + suggested follow-ups at the end. Logs to `sessions/<date>-teach-quiz-<slug>/log.md`. Decline if you don't want it.

Teach is also auto-invoked as a detour from Mock or Tutor or Coach — say "pause — explain X" mid-flow, the walk pauses, the concept gets written, and you're back where you left off. (Detours don't trigger the quiz — that's a standalone-Teach feature.)

The wiki is seeded with 15 concept notes:
- **Foundations:** [cap-theorem](concepts/cap-theorem.md), [sql-vs-nosql](concepts/sql-vs-nosql.md), [sharding](concepts/sharding.md), [replication](concepts/replication.md), [consistent-hashing](concepts/consistent-hashing.md)
- **Messaging & data flow:** [kafka](concepts/kafka.md), [cdc](concepts/cdc.md), [leader-election](concepts/leader-election.md)
- **Reliability patterns:** [rate-limiting](concepts/rate-limiting.md), [circuit-breaker](concepts/circuit-breaker.md), [idempotency-keys](concepts/idempotency-keys.md), [distributed-locks](concepts/distributed-locks.md)
- **Caching & traffic:** [caching-strategies](concepts/caching-strategies.md), [load-balancer](concepts/load-balancer.md), [realtime-transports](concepts/realtime-transports.md)

Add to it as you study — it's the artifact that compounds.

## Company calibration

Pass a company name as the second argument to any mode (or include it in a freeform paste) to apply that company's calibration:

```
/sd-coach mock netflix
/sd-coach tutor google design-url-shortener
/sd-coach drill capacity google
```

Each company file (`companies/<name>.md`) sets:
- A question pool the mode can pick from
- Rubric weights (which scorecard dimensions are emphasized)
- Cultural emphasis the mode reflects in its feedback
- Anti-patterns the mode flags explicitly when it sees them

Seeded: [netflix](companies/netflix.md), [google](companies/google.md), [meta](companies/meta.md), [nuro](companies/nuro.md) (autonomous-delivery / robotics SWE infra — different shape from FAANG; see also [examples/mock-nuro-fleet-telemetry](examples/mock-nuro-fleet-telemetry/)). For any other company name (Amazon, Apple, Stripe, Databricks, etc.) the skill auto-calibrates from cues in the paste / JD, or falls back to FAANG-generic L3–L4 defaults. After a session with auto-calibration, the skill offers to save it as `companies/<name>.md` so it persists.

To pre-add a company manually, copy [companies/_template.md](companies/_template.md).

## Calibration philosophy

Modes score against **L3–L4** specifically — entry-level SDE1 through mid-level SDE2. Each rubric dimension has explicit L3 hit / L3 partial / L3 miss criteria, and the same for L4. The "clears L5+" notes in the rubrics are for awareness only — the modes do not grade against L5.

What this means concretely:
- A "hit" doesn't mean perfect — it means cleared the bar for the level being scored.
- The scorecard tells you whether you cleared the L3 bar AND whether you cleared the L4 bar, separately. You can be L3-hit / L4-partial on the same dimension.
- The "to clear L4" section of the scorecard is the actionable part — specific things to add to your answer, calibrated to your current target.

If you want to study at L5+ later, the rubrics name what would clear that bar in each dimension — but no mode will judge you against it unless you ask.

## Examples

[examples/](examples/) has one synthetic run per mode showing the output shape:
- [Mock — URL shortener](examples/mock-url-shortener/) — partial transcript, Mermaid HLD, full scorecard
- [Tutor — Rate limiter](examples/tutor-rate-limiter/) — shows `[tutor-interrupt]`, `[user-interrupt]`, and `[detour]` annotations in action
- [Drill — Capacity, 5 reps](examples/drill-capacity/) — full 5-rep log with difficulty adjustment
- For Teach examples, the [concepts/](concepts/) wiki itself is the artifact — every note there was written by Teach mode.

These are illustrative — they show what good output should look like before you run your own. Real sessions you generate locally land in `sessions/` (gitignored) and stay private.

## How it works

- `SKILL.md` is the entry point — Claude Code loads it when `/sd-coach` is invoked.
- `SKILL.md` parses the user's input and routes to one of `modes/{mock,tutor,drill,teach}.md`.
- Each mode spec instructs Claude how to behave for that mode and references the relevant rubrics in `rubrics/` and the company calibration in `companies/<name>.md`.
- Mock and Drill write transcripts and scorecards to `sessions/` (gitignored). Teach writes/updates concept notes in `concepts/` (committed).

## Stack

Claude Code skill, markdown + Mermaid. No runtime, no dependencies, no API key.
