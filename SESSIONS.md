# Sessions

Each session is 1–3 hours and ends with a working state + a green push. README is a living document — created in Session 1, updated each session.

---

## Session 1 — Skill skeleton, routing, README v1

- [x] Verify Claude Code skill manifest format (confirmed: `~/.claude/skills/<name>/SKILL.md`, frontmatter with `name`, `description`, `argument-hint`)
- [x] Write `SKILL.md` with the skill manifest
- [x] Top-level mode dispatch in SKILL.md body: parses `$ARGUMENTS`, routes to mode spec, handles freeform paste
- [x] Stub `modes/mock.md`, `modes/tutor.md`, `modes/drill.md`, `modes/teach.md`
- [x] `.gitignore` — `RESUME.md`, `sessions/`, `.DS_Store`, `.env`
- [x] `README.md` v1 — pitch, install via symlink, invocation examples, status table
- [x] Install via symlink: `~/.claude/skills/sd-coach -> /Users/deyvikbhan/develop/sd-coach`
- [x] Static smoke test: frontmatter YAML parses, mode files referenced from SKILL.md exist, .gitignore includes RESUME.md/sessions/.env (caught + fixed unquoted colon in `description` that would have rejected the skill at runtime)

**Done when:** the skill loads cleanly in Claude Code, invocation with no args shows the mode menu, and each of the four mode names routes to its stub.

Completed: 2026-05-07

---

## Session 2 — Mock mode

- [x] `modes/mock.md` — full interviewer prompt with strict in-character constraint
- [x] Phase tracker: scoping → capacity estimation → HLD → API contract → schema → LLD critical path → failure modes, with budgets and transition rules
- [x] Session log writer: instructions for `sessions/<date>-<slug>/transcript.md` + `hld.mmd` + `scorecard.md`
- [x] Scorecard format: cites `rubrics/{hld,lld,capacity-estimation}.md` by dimension, requires verbatim candidate quotes, "no signal" for uncited dimensions (no vibe scoring)
- [x] "Behavior rules NEVER" section: no hints, no teaching, no scope renegotiation, no broken character
- [x] Skeletal rubrics written: `rubrics/hld.md`, `rubrics/lld.md`, `rubrics/capacity-estimation.md` (Session 7 expands)
- [x] Detour handling: mid-mock concept questions get deferred to a `[deferred-questions]` section, offered as Teach mode after scorecard
- [x] Update `README.md` — Mock promoted to "done" in status table; "How Mock works" section added
- [ ] **End-to-end runtime test (on user):** start a real mock with `/sd-coach mock`, confirm transcript + hld.mmd + scorecard get written and the scorecard cites rubrics. Once verified, promote that run to `examples/` in Session 8.

**Done when:** a 45-minute Mock session runs end-to-end on a real prompt, writes transcript + diagram + scorecard to `sessions/`, and the scorecard cites specific rubric dimensions.

Spec complete. Live verification on user.

---

## Session 3 — Tutor mode

- [x] `modes/tutor.md` — Socratic prompt with bidirectional-interrupt protocol
- [x] Tutor-side interrupt trigger list: wrong primitive, unbounded growth, missing back-pressure, wrong consistency model, skipped failure mode, conflated layers
- [x] User-side interrupts: "wait, why X?", "pause — explain X" (detour to Teach), "skip this", "back up"
- [x] Detour-into-Teach hook: pauses, marks `[detour: <topic>]`, runs Teach mode behavior, resumes at decision point
- [x] Session log: `sessions/<date>-tutor-<slug>/tutor-transcript.md` with `[tutor-interrupt]`, `[user-interrupt]`, `[detour]` annotations
- [x] "What this mode does NOT do": no scoring, no clock, no in-character interviewer, no silence through structural mistakes
- [x] Update `README.md` with Tutor usage section + Mock-vs-Tutor differentiation
- [ ] **Runtime verification (on user):** start a real Tutor walk, deliberately make a structural mistake, confirm Tutor interrupts; ask "explain X" mid-flow, confirm detour and resume work

**Done when:** Tutor walks a real design, interrupts me on at least one structural mistake, handles a mid-flow question, and the transcript shows both interrupts and at least one detour stub.

Spec complete. Live verification on user.

---

## Session 4 — Drill mode

- [x] `modes/drill.md` — three tracks: capacity, component, failure-modes
- [x] Track-specific prompt patterns + L3–L4 difficulty floor (easy) and ceiling (hard)
- [x] In-session difficulty adjustment: drop after 2 consecutive misses, step up after 2 consecutive hits, capped at floor/ceiling
- [x] Per-rep flow: prompt → user answer → score → 2–4 sentence model answer (model answer comes AFTER user answers)
- [x] Session log: `sessions/<date>-drill-<track>/log.md` with quoted user answers, model answers, scores, end-of-drill tally and trend
- [x] "Behavior rules NEVER": no >5 reps, no generous "hits", no drift into Tutor, no model-answer-before-user
- [x] Update `README.md` with Drill usage section
- [ ] **Runtime verification (on user):** run a 5-rep drill on the capacity track, confirm difficulty adjusts, log captures all 5

**Done when:** a 5-rep drill run on one track works end-to-end, difficulty visibly adjusts during the run, and the log captures all 5 reps with scores.

Spec complete. Live verification on user.

---

## Session 5 — Teach mode + concept wiki

- [x] `modes/teach.md` — standalone Q&A + mid-Mock/Tutor detour mode
- [x] Wiki write/update logic: read existing `concepts/<slug>.md` first; preserve `> note:` user annotations and `## My notes` sections; append-rather-than-replace on follow-ups (with dated subsections)
- [x] Cross-linking: every note ends in `## Related` linking to real files only (no fabricated links)
- [x] Concept note template: TL;DR / What it is / What it's for / When to reach for it / What it's NOT good at / Alternatives + tradeoffs / Related
- [x] Comparison-question handling: writes `<x>-vs-<y>.md` plus updates both `<x>.md` and `<y>.md`
- [x] Seed 5 concept notes: kafka, consistent-hashing, cap-theorem, cdc, leader-election (all cross-linked)
- [x] Detour wire-through: Mock and Tutor specs already mark `[detour: <topic>]` correctly; teach.md handles the detour path with one-concept-per-detour rule and "Resume?" exit
- [x] Update `README.md` with Teach usage section + links to all 5 seeded concepts
- [ ] **Runtime verification (on user):** ask `/sd-coach teach kafka` and verify the existing note is read + acknowledged; ask a follow-up like "compare to Pulsar" and verify the note gets a dated `### compare to Pulsar — <date>` subsection rather than overwrite

**Done when:** asking the same concept twice updates the existing note instead of duplicating, `concepts/` has 5 seeded notes with cross-links, and a mid-Tutor detour into Teach works end-to-end.

Spec complete + 5 seeded notes shipped. Live verification on user.

---

## Session 6 — Company calibration: Netflix, Google, Meta

- [x] `companies/_template.md` — schema for adding a new company (question pool, rubric weights, cultural emphasis, anti-patterns, L3-vs-L4 expectation)
- [x] `companies/netflix.md` — availability-first, chaos-engineering culture, fallback discipline, multi-region by default; weight-up: scaling, tradeoff-articulation, error-handling, growth
- [x] `companies/google.md` — capacity rigor, fundamentals from first principles, SRE/SLO/tail-latency framing; weight-up: assumptions, math, component-breakdown, tradeoffs
- [x] `companies/meta.md` — product sense first, graph-shape data, fan-out tradeoffs, permission-on-read; weight-up: scoping, data-flow, data-model
- [x] Mode integration is wired through SKILL.md routing (`If a company is named, load companies/<name>.md and apply its rubric weights, question pool, and cultural emphasis`). Each mode references this in its setup section.
- [x] Update `README.md` with a "Company calibration" section + invocation examples
- [ ] **Runtime verification (on user):** run the same Mock prompt with three different companies; diff the scorecards — should be visibly different in emphasis and which dimensions get weight

**Done when:** the same prompt under three different companies produces three visibly different scorecards, and the differences trace back to the company file.

Spec complete. Live verification on user.

---

## Session 7 — Rubrics + L3–L4 calibration pass

- [ ] `rubrics/hld.md` — explicit dimensions (scoping, component breakdown, data flow, scaling, tradeoffs) with L3–L4 line items
- [ ] `rubrics/lld.md` — interface design, data model, error handling, observability hooks
- [ ] `rubrics/capacity-estimation.md` — back-of-envelope steps, when to skip, common traps
- [ ] All modes reference rubrics by file + dimension when scoring/feedback (e.g., "rubrics/hld.md → scaling: ...")
- [ ] L3–L4 floor/ceiling check: re-run a Session 2 prompt; difficulty should match an entry-to-mid-level interview
- [ ] Update `README.md` with a "calibration philosophy" note

**Done when:** every scorecard from every mode cites at least 2 rubric dimensions by file + name, and difficulty floor/ceiling reads as L3–L4-appropriate.

---

## Session 8 — DEMO.md, README polish, sample sessions in `examples/`

- [ ] Move one canonical run of each of the four modes from `sessions/` to `examples/` (cleaned, anonymized — these are the public artifacts)
- [ ] `DEMO.md` — reproducible command sequence for each mode with expected outputs
- [ ] `README.md` polish — pitch, run instructions, four-mode walkthrough each linking to its `examples/` artifact, stack list at bottom
- [ ] Final smoke test: a fresh checkout + skill load + one mock run + one teach run, no edits required
- [ ] Final push

**Done when:** a stranger can read `README.md` + `DEMO.md` and run their first Mock in under 2 minutes, and `examples/` shows one real run per mode.
