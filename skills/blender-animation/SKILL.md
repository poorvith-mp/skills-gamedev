---
name: blender-animation
last_reviewed: 2026-09-06
group: 3D assets
description: >-
  Rig, skin and animate characters and props, and export clean animation sets to an engine. Use
  when rigging skeletons, weight painting, or keyframing animations.
---

# blender-animation

## Core Philosophy
Character and asset animation in Blender for real-time game engines is not about producing a single cinematic render. Game animation is modular, state-driven motion engineering. Animations must blend seamlessly in real time across locomotion states, maintain precise skeletal hierarchy conventions, adhere to bone transform constraints, and export clean, non-destructive FBX/glTF clips with root motion preservation and zero scale distortion.

---

## 4-Step Technical Blender Animation Pipeline

### Step 1: Rigging Architecture & Armature Hierarchy
1. **Skeletal Hierarchy Standards**:
   - Establish an immutable root bone located at world origin `(0, 0, 0)` named `root`.
   - Hierarchy: `root` -> `pelvis` -> `spine_01` -> `spine_02` -> `chest` -> `neck` -> `head`.
   - Limb Suffix Conventions: Strictly use `.L` and `.R` (e.g. `hand.L`, `foot.R`) to enable Blender's native X-Axis Mirroring and Pose Flipping tools.
2. **IK / FK Switching Systems**:
   - Arms and Legs: Build dual Inverse Kinematics (IK) and Forward Kinematics (FK) chains with a custom property slider for smooth blending.
   - Pole Targets: Set up pole target bones for knees and elbows with explicit Pole Angle calibration to prevent joint twisting.

### Step 2: Weight Painting & Skinning Hygiene
1. **Deformation vs Mechanism Bones**:
   - Separate deformation bones (which deform the mesh) from mechanism/control bones (which the animator poses).
   - Prefix deformation bones with `DEF_` and ensure only `DEF_` bones have "Deform" checked in Bone Properties.
2. **Joint Deformation Topology**:
   - Enforce 3 edge loops across all bending joints (elbows, knees, finger knuckles) to prevent mesh collapsing during extreme bends.
   - Normalize vertex weights: Every vertex must have total weights summing strictly to **1.0**, with a maximum of **4 bone influences per vertex** for mobile/console engine compatibility.

### Step 3: Keyframing, Animation Principles & Root Motion
1. **The Core Locomotion Suite**:
   - Deliver complete sets: Idle, Walk, Run, Sprint, Jump Start, Jump Loop (Fall), Jump Land, Dodge/Roll.
   - Maintain 60 FPS timeline calibration; ensure walk and run cycles are frame-perfect loops (Frame 1 pose matches Frame $N$ pose identically).
2. **Root Motion vs In-Place Animation**:
   - *In-Place*: Character stays at `(0, 0, 0)`; engine handles capsule displacement.
   - *Root Motion*: True character physical displacement is keyed into the `root` bone on the ground plane, allowing realistic foot-planting without foot-sliding.

### Step 4: Non-Linear Animation (NLA) & Engine Export
1. **Action Management & NLA Strips**:
   - Push every completed animation into an NLA Action strip; enable "Fake User" (`F`) to prevent Blender from purging unlinked animations on save.
2. **Engine Export Settings (Unity / Unreal)**:
   - Apply All Transforms (`Ctrl+A` -> Rotation & Scale) before export. Ensure armature scale is strictly `(1.0, 1.0, 1.0)`.
   - Export FBX:
     - Main: Selected Objects, Armature + Mesh only.
     - Armature: **Uncheck "Add Leaf Bones"** (prevents useless dummy bones at bone tips).
     - Animation: Bake Animations enabled, Key All Bones, Simplify = 0.0.

---

## Deliverable Format: Character Animation Set Specification (`ANIMATION-SPEC.md`)

```markdown
# Character Animation Set Specification: [Character Rig Name]

## 1. Armature Standards & Skeleton
- **Master Root Bone**: `root` @ (0,0,0)
- **Target Engine**: Unreal Engine 5 (UE5 Mannequin compatible) / Unity URP
- **Max Vertex Weights**: 4 influences / vertex (Normalized to 1.0)
- **Skeletal Scale**: 1.000 uniform scale applied

## 2. Required Animation Action Manifest
| Action Name | Type | Frame Count | Loopable | Root Motion |
|---|---|---|---|---|
| `Hero_Loco_Idle` | Locomotion | 120 frames (60 FPS) | Yes | In-Place |
| `Hero_Loco_Walk_Fwd` | Locomotion | 60 frames (60 FPS) | Yes | Root Motion (`root` bone) |
| `Hero_Loco_Run_Fwd` | Locomotion | 40 frames (60 FPS) | Yes | Root Motion |
| `Hero_Combat_Attack01`| Attack | 45 frames (60 FPS) | No | Root Motion forward 1.2m |
| `Hero_Jump_Land` | Transition | 20 frames (60 FPS) | No | In-Place |

## 3. Export Parameters (Blender FBX)
- Primary Bone Axis: `-Y Forward` | Secondary Bone Axis: `Z Up`
- Add Leaf Bones: **DISABLED**
- Simplify: `0.0`
```

---

## Worked Example: Eliminating Foot-Sliding via Root Motion

- **Problem**: In-place walk cycle caused character boots to slide visibly across the terrain when walking speed didn't match code capsule velocity.
- **Solution**: Re-keyed animation using Root Motion. Keyed forward translation directly onto the `root` ground bone matching foot contact frames.
- **Outcome**: Engine extracted root displacement dynamically; foot-sliding eliminated 100%.

---

## Verification Checklist

- [ ] Armature scale is strictly uniform `(1.0, 1.0, 1.0)` with transforms applied.
- [ ] Only deformation bones (`DEF_`) have vertex weights assigned.
- [ ] Max 4 bone influences per vertex, normalized to 1.0.
- [ ] Export FBX disables "Add Leaf Bones".
- [ ] Looping locomotion animations match start and end frame poses seamlessly.

---

## Anti-Patterns

- **Unapplied Object Scale**: Animating an armature with scale `(0.01, 0.01, 0.01)`, causing models to explode or deform grotesquely upon engine import.
- **Over-Influenced Vertices**: Allowing 8 different bones to influence a single shoulder vertex, destroying mobile GPU skinning performance.
- **Leaf Bones Bloat**: Leaving "Add Leaf Bones" enabled, creating 40 useless dummy bones that clutter engine animation blueprints.
