---
name: anti-cheat
group: Engineering
description: Enforce server authority and defend against speed hacks, memory editing, and exploits. For networking setup, see unity-multiplayer. Use when defending games against exploits or hacks.
---

# anti-cheat

## Core Philosophy
In multiplayer games, the client is fundamentally compromised. The game executable runs in a hostile environment controlled by the user: memory can be modified via Cheat Engine, network packets can be intercepted, and GPU rendering pipelines can be injected with custom shaders. True anti-cheat engineering does not rely on invasive kernel drivers alone; it is built on an uncompromising architectural foundation: **Server Authority**. The client only sends input intentions; the authoritative server simulates physics, validates mechanics, and adjudicates reality.

---

## 4-Step Server-Authoritative Anti-Cheat Architecture

### Step 1: Server Authority & Movement Validation
1. **Never Trust Client Coordinates**:
   - The client must never send absolute positional updates (`player.position = x, y, z`).
   - The client sends raw input vectors (Move forward, Jump, Delta time). The server simulates physics and returns authoritative state.
2. **Speed-Hack & Teleportation Prevention**:
   - Server tracks positional displacement per tick:
     $$\Delta d \le v_{text{max}}  imes \Delta t + \epsilon_{text{threshold}}$$
   - If displacement exceeds maximum legal speed plus latency buffer, snap the player back to their last validated server position and flag anomalous telemetry.
3. **Ray-Swept Collision Bounds**:
   - Verify movement vectors using server-side ray sweeps to prevent clipping through solid geometry ("noclip" cheats).

### Step 2: Combat & Action Validation
1. **Fire Rate & Ammo Accounting**:
   - Track weapon state strictly on the server: cooldown timer between shots, reload animations, magazine capacity.
   - Instantly drop firing requests if the time delta between shots is lower than the weapon's configured firing delay.
2. **Lag Compensation & Rewind Bounds**:
   - For hitscan weapons, the server maintains a circular buffer of past entity positions (typically 1,000ms).
   - When a client claims a hit at timestamp $T$, the server rewinds player hitboxes to timestamp $T$, performs a raycast, and validates line-of-sight.
   - *Security Bound*: Cap maximum backward rewind at **200ms**. If client latency exceeds 200ms, hits are rejected to prevent players behind cover from being shot by lagging clients.

### Step 3: Statistical Heuristics & Aimbot Detection
1. **Angular Velocity & Entropy Analysis**:
   - Human aim exhibits micro-adjustments, tremors, and acceleration/deceleration curves.
   - Aimbots snap instantaneously: angular velocity $ o \infty$, angular acceleration approaches a single-frame delta, and angular path entropy drops to zero.
   - Log server-side telemetry on crosshair angular snap speed; flag players who consistently snap within 1 frame across $> 30^\circ$ angles.

### Step 4: Memory Obfuscation & Heartbeat Integrity
1. **Encrypted In-Memory Variables**:
   - Obfuscate high-value memory addresses (Player Health, Ammo, Gold) using dynamic XOR bitmasks changed every game tick.
2. **Deterministic Simulation Hashes**:
   - Run cryptographic checksums on loaded map geometry and weapon config tables at match start to detect modified local asset files.

---

## Deliverable Format: Anti-Cheat System Specification (`ANTI-CHEAT-SPEC.md`)

```markdown
# Server-Authoritative Anti-Cheat Architecture: [Game Title]

## 1. Authority Model & Tick Rates
- **Server Tick Rate**: 64 Hz (15.625ms per tick)
- **Authority Scope**: Movement, Combat, Inventory, State Machine (100% Server Authoritative)
- **Lag Compensation Window**: Capped at 180ms maximum rewind

## 2. Real-Time Input Validation Rules
| Mechanic | Client Payload | Server Validation Logic | Enforcement Action |
|---|---|---|---|
| Movement | Input Vector + View Angles | Swept-sphere collision + max velocity cap | Hard snapback to last valid pos |
| Fire Weapon | Shoot Event + Target ID | Check ammo count > 0 & cooldown timer expired | Drop packet silently; log warning |
| Reload | Reload Request | Check reload duration >= 2.4s before resetting ammo | Reset reload progress on interrupt |

## 3. Server-Side Heuristic Detection Thresholds
- **Snap Angle**: > 45° angular displacement in <= 1 frame (15.6ms).
- **Hit Ratio**: Headshot ratio > 78% across 10 consecutive matches triggers manual review.
- **Speed Variance**: Accumulated distance delta > 1.15x theoretical max over 3 seconds.
```

---

## Worked Example: Stopping Speed Hacks in a Fast-Paced Shooter

- **Exploit**: Cheaters modified client `Time.timeScale` to move 3x faster than normal players.
- **Server Fix**: Implemented server-side distance delta accumulation over rolling 20-tick windows.
- **Result**: Cheaters were clamped to legal speed caps and rubber-banded backward; exploit neutralized with zero false-positive kicks for high-ping players.

---

## Verification Checklist

- [ ] All positional movement and combat actions adjudicated on the server.
- [ ] Lag compensation rewinds capped at $\le 200text{ms}$.
- [ ] Weapon fire rates, reload timers, and ammo pools tracked server-side.
- [ ] Heuristic telemetry detects unnatural angular snap velocities.
- [ ] Asset checksums verify local map and weapon files before match entry.

---

## Anti-Patterns

- **Client-Authoritative Hits**: Trusting the client when it sends "I hit Player B for 100 damage", inviting trivial memory-injection cheats.
- **Unbounded Lag Compensation**: Letting 500ms ping players shoot people who ran behind concrete walls 2 seconds ago.
- **Client-Only Anti-Cheat**: Relying exclusively on client-side scanning software without server-side validation.
