---
name: playtesting
last_reviewed: 2026-09-06
group: Craft
description: >-
  Structure playtest sessions and sort responses into UX friction, difficulty calibration and
  feature signal. Use when structuring playtest sessions, telemetry, or feedback loops.
---

# playtesting

## Core Philosophy
Playtesting is an empirical research laboratory, not an ego-gratification showcase. The goal of a playtest is never to seek validation or hear friends say "the game is fun." The goal is to rigorously diagnose friction points, player mental model breakdowns, difficulty spikes, and cognitive overload by observing player actions rather than listening to their words. What players *say* is often rationalization; what players *do*—where they die, where they stop moving, what menus they close in confusion—is the only undeniable truth.

---

## 4-Step Playtesting Architecture

### Step 1: Playtest Cohort Segmentation & Protocol Design
1. **The Three Distinct Cohorts**:
   - **First-Time Users (FTUE / Usability Testing)**: Zero genre experience. Tests onboarding clarity, control readability, HUD ergonomics, and camera friction.
   - **Core Genre Target**: Plays similar competitive or narrative titles. Tests mechanical depth, challenge pacing, and core loop retention.
   - **Hardcore / Meta Gamers**: Pushes edge cases, min-maxing, cheese strategies, and economy exploits.
2. **The "Silent Observer" Protocol**:
   - Never guide or instruct the player during the playtest session. If the player is lost for 5 minutes staring at a closed door, take notes—do NOT tell them how to open it.

### Step 2: Triaging Feedback: UX Friction vs Difficulty vs Feature Signal
1. **The Tri-Category Framework**:
   - **UX & Control Friction (Must Fix Immediately)**: Players cannot find the inventory key, misread mission objectives, or confuse decorative background art for interactive platforms.
   - **Difficulty Calibration (Tune Parameters)**: Boss damage output, resource drop rates, checkpoint spacing, ammo scarcity. Tuned with numerical data, not new code.
   - **Feature Signal / Player Solutions (Filter Carefully)**: Players saying *"You should add crafting and multiplayer!"*. Never implement player solutions directly; investigate the underlying symptom (e.g. *"Player felt powerless in combat, so they suggested adding crafting"*).

### Step 3: Quantitative Telemetry Integration
1. **Critical Event Heatmaps**:
   - Hook automated analytics to log:
     - Death coordinates $(X, Y, Z)$ and death cause.
     - Dwell time per room / level segment.
     - Resource bank balances at each major milestone.
     - Quit moment timestamps (Churn coordinates).
2. **Funnel Drop-Off Metric**:
   $$\text{Level Completion Rate} = \frac{\text{Completed Level } N}{\text{Entered Level } N} \times 100$$
   - A drop-off $> 30\%$ at any specific stage indicates a lethal design blocker.

### Step 4: Post-Session Debrief & Semantic Interviewing
1. **Non-Leading Socratic Questions**:
   - *Bad*: "Did you like the shotgun?"
   - *Good*: "Walk me through how you decided which weapon to equip during the courtyard encounter."
   - *Bad*: "Was the puzzle too hard?"
   - *Good*: "What was your goal when you walked into the laboratory room?"

---

## Deliverable Format: Standardized Playtest Observation Matrix

| Player ID | Cohort | Encounter / Level | Observed Behavior (Action) | Stated Reason (Words) | Root Cause Category | Actionable Solution |
|---|---|---|---|---|---|---|
| #P01 | FTUE | Level 1: Ruin Gate | Walked past gate 4 times; attacked locked door with sword | "I didn't know where to go" | UX Friction: Lever lacked visual contrast & lighting | Add rim-light spotlight to the lever and emissive glow |
| #P02 | Core | Boss 1: Golem | Died 6 times on phase 2 overhead smash | "The boss hits too fast to dodge" | Difficulty: Attack startup frame window too short (120ms) | Extend telegraph windup animation from 120ms to 350ms |
| #P03 | Hardcore | Level 2: Arena | Jumped repeatedly on wall geometry to skip fight | "I found a cool shortcut" | Exploit: Missing invisible collision barrier | Add BlockingVolume collider to cliff edge |

---

## Worked Example: Diagnosing a 64% Day 1 Tutorial Churn Spike

- **Problem**: Steam demo playtest telemetry revealed a 64% drop-off within the first 8 minutes of gameplay.
- **Observation**: 12 recorded video sessions showed 10 out of 12 players attempting to double-jump across a chasm before unlocking the double-jump ability, falling into the void, losing currency, and closing the game.
- **Root Cause**: The environmental level design framed the chasm as traversable, but the player did not possess the required mechanic yet.
- **Fix**: Replaced the chasm with a collapsed iron portcullis requiring a power battery located 20 meters away.
- **Outcome**: Level 1 tutorial completion rate surged from 36% to 89% in the subsequent playtest round.

---

## Verification Checklist

- [ ] Playtesters match defined player personas (FTUE, Core, or Hardcore).
- [ ] Observers follow the "Silent Protocol" without offering hints or coaching.
- [ ] Telemetry logs coordinate-based deaths, level dwell times, and quit events.
- [ ] Feedback is categorized into UX Friction, Difficulty Tuning, or Feature Noise.
- [ ] Post-test interview questions are open-ended and non-leading.

---

## Anti-Patterns

- **Leading the Witness**: Standing behind the playtester saying "Press E to open that door" when they get stuck.
- **Defending the Design**: Arguing with a playtester who says "I was confused" by explaining why they should have understood it.
- **Implementing Player-Suggested Features**: Adding complex new subsystems because a tester suggested it, rather than fixing the core root friction.
