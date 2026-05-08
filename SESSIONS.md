# Sessions

Each session is 1–3 hours and ends with a working state + a green push. README is a living document — created in Session 1, updated each session.

---

## Session 1 — Skill skeleton, routing, README v1

- [x] Verify Claude Code skill manifest format (confirmed: `~/.claude/skills/<name>/SKILL.md`, frontmatter with `name`, `description`, `argument-hint`, `arguments`)
- [x] Write `SKILL.md` with the skill manifest (name, description, argument-hint)
- [x] Top-level mode dispatch in SKILL.md body: parses `$ARGUMENTS`, routes to mode spec, handles freeform paste
- [x] Stub `modes/mock.md`, `modes/tutor.md`, `modes/drill.md`, `modes/teach.md`
- [x] `.gitignore` — `RESUME.md`, `sessions/`, `.DS_Store`, `.env`
- [x] `README.md` v1 — pitch, install via symlink, invocation examples, status table
- [x] Install via symlink: `~/.claude/skills/sd-coach -> /Users/deyvikbhan/develop/sd-coach`
- [ ] Runtime smoke test: reload Claude Code, run `/sd-coach`, confirm mode menu appears and each stub routes correctly

**Done when:** the skill loads cleanly in Claude Code, invocation with no args shows the mode menu, and each of the four mode names routes to its stub.

---

## Session 2 — Mock mode

- [ ] `modes/mock.md` — full interviewer prompt with strict in-character constraint
- [ ] Phase tracker: scoping → capacity estimation → HLD → API contract → schema → LLD critical path → failure modes. Each phase has time budget + transition rules.
- [ ] Session log writer: every Mock session writes to `sessions/YYYY-MM-DD-<topic>/transcript.md` + `hld.mmd` + `scorecard.md`
- [ ] Scorecard format: cites rubric dimensions verbatim, quotes my own answers back, no vibe scoring
- [ ] "What this mode does NOT do" section: no hints, no teaching, no scope renegotiation mid-session
- [ ] Run one real Mock end-to-end on a canonical prompt ("Design a URL shortener") — capture as the test artifact
- [ ] Update `README.md` with Mock usage example

**Done when:** a 45-minute Mock session runs end-to-end on a real prompt, writes transcript + diagram + scorecard to `sessions/`, and the scorecard cites specific rubric dimensions.

---

## Session 3 — Tutor mode

- [ ] `modes/tutor.md` — Socratic prompt with bidirectional-interrupt protocol
- [ ] Interrupt rules: Claude interrupts when I make a structural mistake (define what counts), I can interrupt to ask anything mid-flow
- [ ] Detour-into-Teach hook: "pause — explain X" pauses the design, runs Teach mode (Session 5 will fully wire this; stub is fine for now), resumes
- [ ] Session log: `sessions/YYYY-MM-DD-<topic>/tutor-transcript.md` with `[interrupt]` and `[detour]` annotations
- [ ] "What this mode does NOT do": no scoring, no time pressure, no performative interviewer character
- [ ] Run one real Tutor walk on a prompt I haven't fully solved before
- [ ] Update `README.md` with Tutor usage example

**Done when:** Tutor walks a real design, interrupts me on at least one structural mistake, handles a mid-flow question, and the transcript shows both interrupts and at least one detour stub.

---

## Session 4 — Drill mode

- [ ] `modes/drill.md` — three drill tracks: capacity estimation, single-component deep-dive, failure-mode walk
- [ ] Track-specific prompt patterns + difficulty floor/ceiling for L3–L4
- [ ] In-session difficulty adjustment: easier if I miss two in a row, harder if I nail two in a row
- [ ] Session log: `sessions/YYYY-MM-DD-drill-<track>/log.md` with prompt, my answer, the model answer, score
- [ ] Run 5 reps on the capacity-estimation track as the test artifact
- [ ] Update `README.md` with Drill usage example

**Done when:** a 5-rep drill run on one track works end-to-end, difficulty visibly adjusts during the run, and the log captures all 5 reps with scores.

---

## Session 5 — Teach mode + concept wiki

- [ ] `modes/teach.md` — standalone concept Q&A; also invocable mid-Mock/Tutor as a detour
- [ ] Wiki write/update logic: before answering "explain X," read `concepts/<slug>.md` if it exists; update rather than overwrite; preserve my follow-up annotations
- [ ] Cross-linking: each concept note ends with "Related: " links to other concepts in the wiki
- [ ] Concept note template: what it is, what it's for, when to reach for it, what it's NOT good at, alternatives + tradeoffs, related
- [ ] Seed 5 concept notes by running Teach on: Kafka, consistent hashing, CAP theorem, CDC, leader election
- [ ] Wire detour from Mock and Tutor (was stubbed in Sessions 2–3)
- [ ] Update `README.md` with Teach usage example + link to `concepts/`

**Done when:** asking the same concept twice updates the existing note instead of duplicating, `concepts/` has 5 seeded notes with cross-links, and a mid-Tutor detour into Teach works end-to-end.

---

## Session 6 — Company calibration: Netflix, Google, Meta

- [ ] `companies/_template.md` — schema for a company file (rubric weights, question pool, what they push on, cultural notes, anti-patterns at this company)
- [ ] `companies/netflix.md` — emphasis: streaming/availability, microservices boundaries, ops culture
- [ ] `companies/google.md` — emphasis: scale fundamentals, capacity rigor, formal correctness
- [ ] `companies/meta.md` — emphasis: product sense, social-graph patterns, infra-at-scale
- [ ] Mode integration: each mode accepts a company arg and visibly changes behavior (rubric weights, question selection, cultural emphasis in feedback)
- [ ] Acceptance test: run the same Mock prompt with `--company netflix` vs `--company google`, diff the scorecards — must be visibly different in emphasis
- [ ] Update `README.md` with company usage

**Done when:** the same prompt under three different companies produces three visibly different scorecards, and the differences trace back to the company file.

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
