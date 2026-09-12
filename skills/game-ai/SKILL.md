---
name: game-ai
last_reviewed: 2026-09-06
group: Craft
description: >-
  Design NPC behaviour: behaviour trees, state machines, pathfinding, perception and difficulty
  tuning. Use when designing behavior trees, state machines, or enemy AI.
---

# game-ai

## Core Philosophy
Game Artificial Intelligence (AI) is not academic machine learning trying to predict the future or beat human chess grandmasters. Game AI is an entertainment and pacing illusion. An NPC that plays with superhuman accuracy, pixel-perfect aim, and zero hesitation is frustrating and un-fun to play against. Game AI must provide clear telegraphing, readable tactical decision-making, believable sensory perception, and dynamic difficulty calibration that gives players the thrilling sensation of outsmarting a competent opponent.

---

## 4-Step Game AI Architecture

### Step 1: Decision-Making Architectures: Behavior Trees vs FSMs
1. **Behavior Trees (The Industry Standard for Combat AI)**:
   - Built on hierarchical nodes evaluated from left-to-right:
     - *Composites*:
       - **Sequence ($ o$)**: Executes children sequentially; fails if any child fails (e.g. *Has Weapon* $ o$ *Reload* $ o$ *Aim*).
       - **Selector ($?$)**: Executes children until one succeeds (e.g. *Flee If Low Health* $?$ *Melee Attack* $?$ *Seek Cover*).
     - *Decorators*: Conditional gates (e.g. *Invert, Time Limit, Cooldown*).
     - *Leaves / Tasks*: Concrete executable actions (e.g. *MoveToPosition, PlayAnimation*).
2. **Finite State Machines (FSMs)**:
   - Best for simple ambient creatures or discrete bosses (States: *Patrol, Investigate, Chase, Attack, Flee*).

### Step 2: Sensory Perception Systems (Sight & Sound)
1. **Vision Cones (Sight Perception)**:
   - Evaluate target visibility using 3 gates:
     - *Distance Check*: Is player within sight radius ($D \le R_{text{max}}$)?
     - *Field of View Angle*: Is the dot product of enemy forward vector and direction-to-player $\ge \cos( heta / 2)$?
     - *Line of Sight (Raycast)*: Does a physics raycast from enemy eyes to player torso hit world geometry or the player?
2. **Acoustic Perception (Hearing)**:
   - Propagate sound events through the game world (e.g. Gunshot = 40m radius; Footsteps = 8m radius).
   - Alerted NPCs transition to an *Investigate* state directed toward the sound origin.

### Step 3: NavMesh Pathfinding & Tactical Flanking
1. **A* Pathfinding on Navigation Meshes**:
   - NPCs query pre-baked NavMeshes for path traversal.
   - *Reciprocal Velocity Obstacles (RVO / ORCA)*: Implement local collision avoidance so groups of 5 enemies don't march in an identical single file or jam doorway entrances.
2. **Tactical Cover Finding & Flanking**:
   - Query Environmental Query Systems (EQS): Sample candidate points around the player, score points based on cover geometry angle, distance, and line-of-sight protection.

### Step 4: Telegraphing & The Illusion of Competence
1. **Telegraphing Intent**:
   - Before executing a lethal attack, NPCs must telegraph intent through audio cues ("Flanking left!", "Grenade out!"), wind-up animations, or laser sight lines.
2. **The "Stormtrooper Effect" (Controlled Incompetence)**:
   - First shot fired at an unaware player should intentionally miss by 1 meter to alert the player and create adrenaline before dealing real damage.

---

## Deliverable Format: Enemy AI Behavior Tree Specification (`ENEMY-AI-SPEC.md`)

```markdown
# Combat Enemy AI Behavior Tree Specification: [NPC Archetype]

## 1. Perception Parameters
- **Sight Distance**: 25 meters (Day) / 14 meters (Night)
- **Field of View Angle**: 110 degrees
- **Hearing Radius**: Gunfire = 50m | Running Footsteps = 12m | Crouch Walk = 0m (Silent)
- **Reaction Time Buffer**: 350ms delay between target acquisition and attack trigger

## 2. Master Behavior Tree Hierarchy
```
Root (Selector)
├── [Decorator: Health < 20%] -> Sequence (Emergency Flee)
│   ├── FindCoverAwayFromPlayer()
│   ├── MoveToCover()
│   └── PlayAnimation("Heal")
├── [Decorator: CanSeePlayer == True] -> Selector (Combat Branch)
│   ├── [Decorator: InMeleeRange] -> Sequence (Melee Strike)
│   │   ├── PlayAudioBark("Charge!")
│   │   ├── TelegraphWindup(400ms)
│   │   └── ExecuteMeleeDamage()
│   └── Sequence (Ranged Engagement)
│       ├── Selector (Take Cover or Flank)
│       │   ├── [Decorator: InCover] -> AimAtPlayer()
│       │   └── MoveToFlankPosition()
│       └── FireBurst(3_rounds)
└── Sequence (Patrol / Idle)
    ├── PickNextPatrolWaypoint()
    ├── MoveToWaypoint()
    └── WaitRandom(2s, 5s)
```

## 3. Audio Barks & Player Feedback
- **On Spotting Player**: *"Contact! Nine o'clock!"*
- **On Grenade Toss**: *"Frag out! Take cover!"*
```

---

## Worked Example: Fixing "Unfair" Sniper Enemy AI

- **Problem**: Playtesters hated a sniper enemy because it instantly killed players across the map without warning.
- **Remediation**: Added a visible red laser sight line sweeping across the ground for 2.0 seconds before firing, paired with a distinct audio charge-up whine.
- **Outcome**: Player deaths felt fair and avoidable; playtest satisfaction scores jumped from 32% to 88%.

---

## Verification Checklist

- [ ] Perception system validates distance, FOV cone, and physics raycast line-of-sight.
- [ ] Behavior tree prioritizes survival/flee states before offensive actions.
- [ ] All lethal attacks include readable telegraphing animations or audio barks.
- [ ] NavMesh agents implement local avoidance (RVO) to prevent doorway crowding.
- [ ] Enemy reaction times include realistic human-like delay buffers (200–400ms).

---

## Anti-Patterns

- **Omniscient AI**: Letting enemies know player coordinates through solid walls without perception checks.
- **Frame-Zero Snapping**: Enemies pivoting $180^\circ$ and firing with 100% accuracy in a single frame.
- **The Single-File Lemmings**: Having 8 enemies march in a straight line through a single hallway to be slaughtered one by one.
