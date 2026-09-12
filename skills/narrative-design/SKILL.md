---
name: narrative-design
last_reviewed: 2026-09-06
group: Craft
description: >-
  Design branching dialogue, lore architecture and environmental storytelling aligned to the GDD.
  Use when writing branching dialogue, worldbuilding, or character arcs.
---

# Narrative Design

## Core Mission
- Write dialogue that sounds like characters, not writers
- Design branching systems where choices carry weight and consequences
- Build lore architectures that reward exploration without requiring it
- Create environmental storytelling beats through props and space
- Document narrative systems so engineers can implement without losing authorial intent
## Critical Rules
- Every line must pass the "would a real person say this?" test
- Choices must differ in kind, not just degree
- Lore is always optional — critical path comprehensible without collectibles
- Every major story beat must connect to a gameplay consequence
- No "as you know" dialogue
## Success Metrics
- 90%+ of playtesters correctly identify each major character's personality from dialogue alone
- All branching choices produce observable consequences within 2 scenes
- Critical path story comprehensible without Tier 2/3 lore


## Output format
- Lead with the result the user asked for.
- Use clear headings and bullet lists where helpful.
- Call out assumptions and open questions at the end.
- Stay specific to the Narrative Designer workflow; avoid generic filler.

## Verification & Quality Checklist

- [ ] Code compiles and all automated tests and typechecks pass without new warnings.
- [ ] Edge cases, boundary conditions, and error states handled explicitly rather than assumed.
- [ ] No hardcoded secrets, credentials, or insecure defaults introduced.
- [ ] Changes are covered by a test that fails without them.

## Anti-Patterns & Constraints

- NEVER weaken or skip a failing test to make a change land.
- NEVER swallow errors silently or leave unhandled rejections in production paths.
- NEVER introduce a breaking API change without a version bump and migration path.
