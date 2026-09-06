---
name: game-performance
group: Engineering
description: >-
  Hit frame budget: draw calls, batching, GC spikes, LOD strategy and in-engine profiling. Use
  when profiling frame times, draw calls, GPU bottlenecks, or memory.
---

# game-performance

## Core Philosophy
Game performance optimization is not randomly running Unity's profiler the night before release and guessing which scripts to tweak. Real-time rendering is an unyielding mathematical budget. At 60 frames per second, the entire game engine—CPU game logic, AI pathfinding, physics simulation, audio processing, GPU vertex transformation, rasterization, and post-processing—has strictly **16.66 milliseconds** to finish. At 120 FPS, that budget shrinks to **8.33 milliseconds**. Hitting frame budgets requires identifying bottlenecks, managing garbage collection allocations, and optimizing draw calls.

---

## 4-Step Real-Time Performance & Profiling Architecture

### Step 1: The Frame Budget Breakdown (16.66ms @ 60 FPS)
1. **The Dual-Engine Budget (CPU vs GPU)**:
   - *CPU Budget (Target: $\le 10text{ms}$)*:
     - Game Logic & Scripts: $\le 4text{ms}$
     - Physics Simulation: $\le 3text{ms}$
     - Render Thread / Draw Call Preparation: $\le 3text{ms}$
   - *GPU Budget (Target: $\le 14text{ms}$)*:
     - Shadow Maps & Geometry Passes: $\le 4text{ms}$
     - Shading & Lighting Passes: $\le 6text{ms}$
     - Post-Processing & UI: $\le 4text{ms}$
2. **Diagnose the Bottleneck First**:
   - *CPU-Bound*: GPU utilization is $< 85\%$; frame time drops when adding AI entities or physics objects.
   - *GPU-Bound*: GPU utilization is $99–100\%$; frame time drops when increasing screen resolution, shadow quality, or particle density.

### Step 2: CPU Optimization & Garbage Collection (GC) Defense
1. **Zero-Allocation in the Hot Loop**:
   - In Unity / C#, executing `new` inside `Update()`, `FixedUpdate()`, or `LateUpdate()` allocates heap memory, triggering non-deterministic Garbage Collection spikes that cause visible micro-stutters.
   - *Golden Rules*:
     - Cache component references in `Awake()` or `Start()` (never call `GetComponent()` in `Update`).
     - Pre-allocate reusable collections (`List<T>`, arrays); use `list.Clear()` instead of `new List<T>()`.
     - Implement **Object Pooling** for all ephemeral entities (bullets, particles, sound effects, damage numbers).

### Step 3: GPU Optimization: Draw Calls, Batching & Overdraw
1. **Draw Call Consolidation**:
   - Target: Maximum **500–1,000 draw calls** per frame on mobile; **2,000–3,000** on PC/consoles.
   - Techniques:
     - *Static Batching*: Combine non-moving geometry sharing materials into single meshes.
     - *GPU Instancing*: Render 1,000 identical meshes (trees, grass, projectiles) in a single draw call.
     - *Texture Atlasing*: Combine multiple prop textures into a single 4K atlas to share a single material.
2. **Overdraw & Fill-Rate Defense**:
   - Transparent particles (smoke, fire, fog) stacked on top of each other force the GPU to shade the same screen pixel 10 times.
   - Enable Occlusion Culling to prevent rendering objects hidden behind solid walls.

### Step 4: Level of Detail (LOD) & Distance Culling
1. **The 3-Tier LOD Ramp**:
   - *LOD 0 (0–15m)*: 100% mesh triangles (Hero detail).
   - *LOD 1 (15–40m)*: 40% mesh triangles (Simplified bevels, removed micro-props).
   - *LOD 2 (> 40m)*: 10% mesh triangles (Basic silhouette).
   - *Billboard / Impostor (> 100m)*: 2D flat quad facing camera.

---

## Deliverable Format: Performance Budget & Profiling Report (`PERFORMANCE-BUDGET.md`)

```markdown
# Real-Time Game Performance Budget & Profiling Audit: [Scene Name]
*Target Hardware: Steam Deck / Mid-Spec PC (GTX 1660 / Ryzen 5) | Target: 60 FPS (16.6ms)*

## 1. Frame Time Budget Allocation
| Subsystem | Budget (ms) | Actual Measured (ms) | Status | Primary Culprit |
|---|---|---|---|---|
| CPU Game Logic | 4.0ms | 3.2ms | **Pass** | Clean cached lookups |
| CPU Physics | 3.0ms | 4.8ms | **FAIL** | 200 unbatched rigidbodies colliding |
| CPU Render Thread | 3.0ms | 2.4ms | **Pass** | 820 draw calls |
| GPU Base Pass | 6.0ms | 5.1ms | **Pass** | PBR materials |
| GPU Translucency / FX | 3.0ms | 6.4ms | **FAIL** | Particle overdraw on explosions |
| **Total Frame Time** | **16.6ms** | **21.9ms (45 FPS)**| **FAIL** | Physics + Particle Overdraw |

## 2. Identified Performance Bottlenecks & Fixes
1. **Physics Stutter**: 200 physics debris parts were querying continuous mesh colliders.
   - *Fix*: Switched debris to simple sphere colliders; set kinematic sleep threshold to 0.2s.
2. **Explosion Overdraw**: Smoke particle system spawned 400 overlapping transparent quads.
   - *Fix*: Reduced particle count to 40; used customized cut-out mesh particles to eliminate transparent fill-rate waste.

## 3. Post-Optimization Verification
- **New Total Frame Time**: **13.8ms (72 FPS stable)**. Zero GC allocation in hot loops.
```

---

## Worked Example: Eliminating Garbage Collection Spikes in Unity

- **Symptom**: Game suffered a 60ms freeze every 8 seconds during heavy combat.
- **Profiler Trace**: Identified that spawning bullet impacts allocated 45KB of heap per frame, triggering Unity's garbage collector.
- **Remediation**: Implemented a generic pre-allocated `ObjectPool<T>` storing 100 reusable bullet impacts.
- **Outcome**: GC allocations dropped to 0 B per frame in combat; frame rate locked at a solid 60 FPS.

---

## Verification Checklist

- [ ] Total frame time stays under 16.66ms (60 FPS) or 8.33ms (120 FPS) on target baseline hardware.
- [ ] Hot loop methods (`Update()`, `FixedUpdate()`) generate strictly **0 B of GC allocation**.
- [ ] Total draw calls stay within platform limits (<= 1,000 mobile / <= 3,000 PC).
- [ ] Object pooling implemented for all high-frequency entities (projectiles, particles).
- [ ] Level of Detail (LOD) and Occlusion Culling configured and verified.

---

## Anti-Patterns

- **Optimizing in the Dark**: Guessing what is slow without running a frame profiler (Unity Profiler, Unreal Insights, RenderDoc).
- **`GetComponent()` in `Update()`**: Calling expensive scene hierarchy searches 60 times per second per entity.
- **Uncontrolled Alpha Blending**: Spawning 1,000 giant transparent smoke particles covering the entire screen.
