# sd-coach

A personal Claude Code skill for end-to-end system design interview prep. Four modes (Mock, Tutor, Drill, Teach), FAANG-calibrated rubrics for L3–L4 SWE roles, and a personal concept wiki that grows as you study.

## Pitch

System design prep tools either grade transcripts (no spatial reasoning) or stop at HLD (no LLD continuity), and most "company-specific" calibration is flavor text. `sd-coach` lives where I already work — Claude Code — and treats each session as a real artifact: HLD diagrams in Mermaid, an OpenAPI sketch, a schema, one critical-path module written as real code, and a failure-mode walk. Concepts I detour into get written to a personal wiki that compounds across sessions.

## Goals

- Practice end-to-end system design without leaving Claude Code (no API key, no hosted SaaS).
- Four modes: full mock, Socratic tutor, rapid drills, standalone concept teaching.
- Company calibration that actually changes interviewer behavior — not just a sticker on the rubric.
- Build a committed `concepts/` wiki of system design fundamentals tuned by my own follow-up questions.
- Keep session artifacts (diagrams, schemas, code) so I can reference them in real interviews.

## Non-goals

- Not a SaaS, not a hosted tool, not a product. No marketing surface.
- Not aiming for L5+ Staff or principal-level content. L3–L4 difficulty floor and ceiling.
- Not a comprehensive company database. Three seeded (Netflix, Google, Meta) + a template.
- No completionism on `concepts/`. Notes get written when I detour into a topic, not pre-populated.
- No runtime, no servers, no scheduled jobs.

## Stack

- Claude Code skill (markdown-driven, `SKILL.md` manifest)
- Mermaid for diagrams, OpenAPI sketches in YAML, schemas in plain SQL
- Plain markdown for everything else (sessions, concepts, rubrics, companies)
- No dependencies

## Scope

1-week project. 8 sessions, 1–3 hours each. Each session ends with a working state and a green push.

## Modes (spec)

- **Mock** — 45-min in-character interviewer sim. Phases: scoping → capacity → HLD → API → schema → LLD critical path → failure modes. No hints. Scorecard at end.
- **Tutor** — Socratic walk through a design. Bidirectional interrupts (Claude interrupts when I go wrong; I interrupt to ask anything). Can detour into Teach mid-flow without losing place.
- **Drill** — rapid small reps in one of three tracks: capacity estimation, single-component deep-dive, failure-mode walk. 5 reps per drill, difficulty adjusts within session.
- **Teach** — concept Q&A. Standalone or invoked mid-session. Reads existing `concepts/<topic>.md` before answering, prefers updating over duplicating. Cross-links related concepts.

## Folder layout (target)

```
sd-coach/
├── SKILL.md                 # Claude Code skill entry
├── PROJECT.md
├── SESSIONS.md
├── README.md                # living document, updated each session
├── DEMO.md
├── modes/
│   ├── mock.md
│   ├── tutor.md
│   ├── drill.md
│   └── teach.md
├── companies/
│   ├── _template.md
│   ├── netflix.md
│   ├── google.md
│   └── meta.md
├── rubrics/
│   ├── hld.md
│   ├── lld.md
│   └── capacity-estimation.md
├── concepts/                # the personal wiki — committed
├── examples/                # one canonical sample session per mode, committed
├── sessions/                # gitignored — my actual practice runs
└── .gitignore
```

## Risks / unknowns

- **Skill loading semantics.** Claude Code skill format (manifest fields, where this lives — project-local vs `~/.claude/skills/`) needs to be verified in Session 1 before the rest assumes it works.
- **Rubric quality.** "FAANG calibration" is performative if rubrics aren't sharp. Mitigation: each rubric file lists specific line items each mode must cite verbatim in feedback.
- **Concept wiki sprawl.** Without discipline, `concepts/` becomes a junk drawer. Mitigation: Teach mode reads existing notes first, prefers updates, cross-links.
- **Mode prompt drift.** Mock mode could turn into Tutor mode if the prompt isn't strict enough. Mitigation: each mode has a "what this mode does NOT do" section in its spec.
- **Self-honesty.** I'll be tempted to grade myself generously. Mitigation: Mock scorecard cites specific rubric dimensions and quotes my own answers back; no vibe scoring.

## Target resume bullet

> Built a personal system design interview coach as a Claude Code skill: four practice modes (mock, Socratic tutor, drill, concept teaching), FAANG-calibrated rubrics for L3–L4 SWE roles, end-to-end coverage from capacity estimation through HLD, API, schema, and critical-path LLD. Used through [measure: # mock sessions] during interview prep, generating a personal concept wiki of [measure: # entries].
