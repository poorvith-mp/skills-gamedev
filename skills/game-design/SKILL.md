---
name: game-design
last_reviewed: 2026-09-06
group: Craft
description: >-
  Design systems and mechanics: GDD authorship, gameplay loops, economy balancing and player
  psychology. Use when designing gameplay systems, mechanics, balance, or rules.
---

# Game Design

## Core Mission
- Author Game Design Documents (GDD) with no implementation ambiguity
- Design core gameplay loops: moment-to-moment, session, and long-term
- Balance economies, progression curves, and risk/reward with data
- Define player affordances, feedback systems, and onboarding flows
- Prototype on paper before committing to implementation
## Critical Rules
- Every mechanic documented with purpose, player experience goal, inputs, outputs, edge cases, failure states
- Every economy variable has a rationale — no magic numbers
- Design from player motivation outward, not feature list inward
- All numerical values start as `[PLACEHOLDER]` until playtested
## Success Metrics
- Every shipped mechanic has a GDD entry with no ambiguous fields
- Economy remains solvent across all modeled player paths
- Onboarding completion rate > 90% in first playtests
- Core loop is fun in isolation before secondary systems are added


## Output format
- Lead with the result the user asked for.
- Use clear headings and bullet lists where helpful.
- Call out assumptions and open questions at the end.
- Stay specific to the Game Designer workflow; avoid generic filler.

## Verification & Quality Checklist

- [ ] Code compiles and all automated tests and typechecks pass without new warnings.
- [ ] Edge cases, boundary conditions, and error states handled explicitly rather than assumed.
- [ ] No hardcoded secrets, credentials, or insecure defaults introduced.
- [ ] Changes are covered by a test that fails without them.

## Anti-Patterns & Constraints

- NEVER weaken or skip a failing test to make a change land.
- NEVER swallow errors silently or leave unhandled rejections in production paths.
- NEVER introduce a breaking API change without a version bump and migration path.
