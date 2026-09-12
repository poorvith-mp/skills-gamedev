---
name: production-planning
last_reviewed: 2026-09-06
group: Plan
description: >-
  Plan the build: vertical slice, milestones, scope cuts, and what actually ships in the first
  playable. Use when planning milestones, scoping features, or building roadmaps.
---

# production-planning

## Core Philosophy
Game development is the most chaotic engineering discipline in tech because creative fun cannot be scheduled on a Gantt chart. Traditional waterfall planning leads to catastrophic death marches, while unstructured agile causes endless scope creep where games remain in development for 7 years without shipping. Successful game production operates on rigorous phased gating: moving from a brutal Proof of Concept to a polished Vertical Slice, enforcing aggressive scope triage, and defining an unshakeable Definition of Done for First Playable milestones.

---

## 4-Step Game Production Framework

### Step 1: The Phased Milestone Lifecycle
1. **Prototype / Core Loop Validation (Months 1-3)**:
   - *Goal*: Prove the core interaction is mechanically fun with gray-box geometry and zero final art. If the movement and combat do not feel satisfying with untextured cubes, no amount of 4K graphics will save the game.
2. **Vertical Slice (Months 4-7)**:
   - *Goal*: Build 10-15 minutes of final-quality gameplay containing all systems: final art, shaders, combat, UI, audio, save/load, and localization pipeline. This proves the production budget and velocity.
3. **Production / Content Scalpel (Months 8-16)**:
   - *Goal*: Factory-line asset and level generation using established Vertical Slice pipelines.
4. **Alpha $\to$ Beta $\to$ Gold Master (Months 17-20)**:
   - *Alpha*: Feature complete. Zero new systems allowed.
   - *Beta*: Content complete. Bug fixing, balancing, and optimization only.
   - *Gold*: Zero blocker/crash bugs. Console platform certification compliance.

### Step 2: The Vertical Slice Audit Criteria
1. **Vertical Slice Must Include**:
   - 1 complete, polished level/mission.
   - Fully animated player character and minimum 2 enemy archetypes.
   - Polished HUD, inventory, pause menu, and audio mix.
   - Profiler telemetry demonstrating stable frame budget (60 FPS on target hardware).
2. **The "Throwaway Code" Purge**:
   - Strip all prototype hacks and spaghetti code before beginning full production.

### Step 3: Scope Triage: The MoSCoW Feature Razor
1. **Must Have (Core Pillar)**:
   - Features without which the core loop collapses (e.g. gun recoil, cover system, enemy AI).
2. **Should Have (High Value)**:
   - Deepens gameplay but game could launch without it (e.g. weapon crafting, dynamic weather).
3. **Could Have (Scope Cut Candidates)**:
   - First to be chopped when schedule slips: photo mode, fishing mini-game, branching dialogue trees.
4. **Won't Have (Post-Launch / Sequel)**:
   - Multiplayer co-op in a planned single-player game, procedural world generation.

### Step 4: Velocity Tracking & Burndown Metrics
1. **Asset Velocity Calculation**:
   $$\text{Sprint Asset Velocity} = \frac{\text{Completed Game-Ready Assets}}{\text{Target Asset Quota}}$$
2. **Scope Contingency Rule**:
   - Always budget **25% buffer time** before certification and submission dates for unanticipated console platform compliance (TRC/TCR) failures.

---

## Deliverable Format: Milestone Roadmap & Scope Triage Ledger

```markdown
# Project: "Aetheria" Production Roadmap

## Milestone Gates:
- [x] M1: Graybox Core Loop (Movement + Melee Combat) — Approved Week 6
- [x] M2: First Playable (1 Boss Arena + Graybox Telemetry) — Approved Week 14
- [ ] M3: Vertical Slice (15 Min Final Level + 60 FPS Target) — Deadline: Week 26
- [ ] M4: Alpha Gate (Feature Complete, 4 Levels Playable) — Deadline: Week 42
- [ ] M5: Beta Gate (Content Complete, Zero P1 Bugs) — Deadline: Week 54

## Scope Cut Contingency Tiers:
| Tier | Feature Name | Estimated Dev Weeks | Cut Trigger Date |
|---|---|---|---|
| Tier 1 | Dynamic Day/Night Lighting Cycle | 4 Weeks | If M3 delayed by > 2 weeks |
| Tier 2 | Companion Pet AI System | 6 Weeks | If Alpha milestone slips by 3 weeks |
| Tier 3 | Weapon Dye Customization UI | 2 Weeks | Immediate cut if console cert fails |
```

---

## Worked Example: Emergency Scope Cut to Save a 6-Month Slipping Release

- **Context**: An action-adventure studio was 14 weeks behind schedule entering Alpha. The team was attempting to build 12 distinct weapon classes and 4 open-world vehicle types.
- **Intervention**: The production lead applied the MoSCoW razor: slashed the 4 vehicle classes entirely (saving 18 weeks of physics engineering and animation), and consolidated the 12 weapon classes into 5 highly polished archetypes.
- **Outcome**: The team recovered 12 weeks of schedule, eliminated mandatory weekend crunch, and hit console certification on the first submission attempt.

---

## Verification Checklist

- [ ] Every milestone has an objective, non-negotiable Definition of Done.
- [ ] Prototype phase verifies gameplay fun before art asset production begins.
- [ ] Vertical Slice benchmarks real production velocity and frame budgets.
- [ ] Pre-determined Scope Cut list exists before delays occur.
- [ ] Minimum 25% schedule buffer allocated for console certification (Sony TRC / Microsoft TCR).

---

## Anti-Patterns

- **Starting Full Production on Unfun Graybox**: Investing $500K in 3D character art while the movement mechanics still feel clunky and unresponsive.
- **Endless Prototyping (Feature Creep)**: Adding new mechanics every week based on whatever game the creative director played over the weekend.
- **Zero Scope Cut Contingency**: Treating every single planned feature as a sacred requirement until the studio runs out of runway.
