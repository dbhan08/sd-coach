# <Company> calibration

Template for adding a new company to `companies/`. Copy this file to `companies/<name>.md` and fill in.

When a mode is invoked with a company name, this file is loaded and applied to:
- The question pool the mode picks from
- Rubric weights in the scorecard (which dimensions matter more)
- Cultural emphasis in feedback (what to push on)
- Anti-patterns to flag explicitly during the session

Keep this file ≤150 lines. Specificity beats completeness.

## Question pool (L3–L4)

Problems this company is known to ask, calibrated to entry/mid-level. Avoid Staff+ prompts.

- <problem>
- <problem>

## Rubric weights

For the scorecard, weight these dimensions higher or lower than the default. The mode multiplies the relevant rubric scores by this when summarizing.

- **weight up:** `rubrics/hld.md → <dimension>`, `rubrics/lld.md → <dimension>`, ...
- **weight down:** `rubrics/<file> → <dimension>`, ...

## Cultural emphasis

What this company pushes on in interviews, in 2–4 paragraphs. Specific stack/culture notes (Netflix's chaos engineering, Google's SLO framing, Meta's product sense) belong here.

## Anti-patterns (flag explicitly during sessions)

Things this company specifically dings. The mode should call these out by name when seen.

- <anti-pattern> — <why it's specifically bad here>

## L3 vs L4 expectation

One paragraph each. What clears the bar at this company at each level.

**L3 (entry):**
- ...

**L4 (mid):**
- ...

## Notes (optional)

Any extra context, recent shifts in interview style, or known interviewer preferences. Update as you learn more.
