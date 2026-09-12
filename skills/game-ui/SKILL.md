---
name: game-ui
last_reviewed: 2026-09-06
group: Craft
description: >-
  Design HUDs, menus and diegetic interfaces with controller navigation and readability at TV
  distance. Use when designing HUDs, inventory menus, or controller-first interfaces.
---

# game-ui

## Core Philosophy
User Interface (UI) design for video games is fundamentally different from web or SaaS design. Game UI exists in service of immersion, situational awareness, and gameplay flow. A cluttered, opaque web-style dashboard pasted over a 3D game world shatters immersion. Game UI requires strict classification across diegetic and non-diegetic layers, controller-first spatial navigation (the 10-foot television rule), rapid glanceability during high-stress combat, and responsive resolution scaling.

---

## 4-Step Game UI & HUD Architecture

### Step 1: The 4 Interface Immersion Layers
1. **UI Layer Classification**:
   - *Diegetic UI*: Exists inside the physical 3D game world and is experienced by the character (e.g. Dead Space holographic health spine, Pip-Boy wrist computer, physical map held in character hands).
   - *Non-Diegetic UI*: Traditional 2D screen overlay completely invisible to the character (e.g. Health bar at top-left, floating minimap, pause menu).
   - *Spatial UI*: UI elements positioned in the 3D world but invisible to the character (e.g. floating waypoint markers over mission objectives, player name tags).
   - *Meta UI*: Visual effects representing character state on screen (e.g. blood splatters on screen when injured, water droplets on camera lens).

### Step 2: The 10-Foot Television Rule & TV Safe Zones
1. **Console & TV Legibility (The 10-Foot Rule)**:
   - A player sitting 10 feet away from a 55-inch television cannot read 12pt web fonts.
   - *Minimum Font Sizing*: Body text must be **at least 24pt to 28pt** on a 1080p canvas (scaled up proportionally for 4K).
2. **Action Safe & Title Safe Margins**:
   - Television bezels crop screen edges (overscan). Keep all critical HUD elements (health, ammo, minimap) within the **Title Safe Zone** (minimum **10% margin** from screen edges).

### Step 3: Controller-First Spatial Navigation
1. **Deterministic D-Pad / Thumbstick Focus**:
   - Gamepad navigation does not have a mouse cursor. Moving the analog stick must jump focus deterministically to the next spatial neighbor.
2. **Visual Focus Feedback**:
   - The currently focused menu item must be unmistakably obvious: high-contrast outline border, subtle scale-up (1.05x), and distinct audio navigation clicks.
3. **Dedicated Gamepad Button Legends**:
   - Always display controller button prompts on screen matching the active hardware (e.g. `(A) Select`, `(B) Back`, `(X) Reload`). Automatically swap prompt icons between Xbox, PlayStation, and Keyboard when input devices switch.

### Step 4: Combat Glanceability & HUD Hierarchy
1. **The F-Pattern Combat Glance**:
   - In high-speed combat, players have $< 100text{ms}$ to check vital signs.
   - Anchor vital telemetry (Health, Shield, Primary Ammo) close to the screen periphery or clustered near the center reticle.
   - Use high-contrast color coding: Green/Blue (Shields), Red (Low Health), Flashing Amber (Low Ammo).

---

## Deliverable Format: Game UI / HUD Specification (`GAME-UI-SPEC.md`)

```markdown
# Game Interface & HUD Specification: [Game Title]

## 1. UI Layer Taxonomy & Immersion Strategy
- **Combat HUD**: Non-Diegetic minimalist HUD (Fades out when out of combat)
- **Inventory System**: Spatial Diegetic backpack inspection
- **World Waypoints**: Spatial floating 3D markers with distance meters

## 2. Television Safe Zones & Resolution Scaling
- **Reference Canvas**: 1920x1080 (Scales dynamically via UI Canvas Scaler to 4K)
- **Title Safe Margin**: 10% inset (96px horizontal, 54px vertical)
- **Minimum Body Typography**: 26pt bold font
- **Color Contrast**: All HUD text uses 2px dark drop shadow (`rgba(0,0,0,0.85)`) to ensure legibility against bright skyboxes.

## 3. Controller Navigation Map (Main Menu / Inventory)
| Input Button | Action Triggered | Visual Feedback | Audio Cue |
|---|---|---|---|
| D-Pad Up / Down | Jump focus to neighbor | 1.05x scale + 2px gold border | `ui_focus_move.wav` |
| Face Button South (A / Cross) | Select / Confirm | Button depress + white flash | `ui_select_confirm.wav` |
| Face Button East (B / Circle) | Back / Close Menu | Instant screen pop | `ui_cancel_back.wav` |
| Left Trigger / Right Trigger | Switch inventory tabs | Sliding tab indicator | `ui_tab_switch.wav` |

## 4. Combat HUD Glancing Layout
- **Top-Left**: Player Level & Team Health Bars
- **Bottom-Right**: Active Weapon Silhouette, Magazine Counter (Huge 48pt text), Grenade icons
- **Bottom-Center**: Ultimate Ability readiness ring (Glows when 100% charged)
```

---

## Worked Example: Fixing Tiny Text in a Console Port

- **Problem**: PC game was ported to Steam Deck and Xbox; playtesters complained they could not read weapon stats or inventory descriptions.
- **Remediation**: Implemented a responsive UI scaling mode for handheld/TV screens. Increased minimum body text from 14pt to 26pt; added a high-contrast dark backing card behind weapon comparison tooltips.
- **Outcome**: Steam Deck verified status achieved; zero negative reviews regarding text legibility.

---

## Verification Checklist

- [ ] All critical HUD telemetry placed within the 10% Title Safe boundary.
- [ ] Typography adheres to the 10-foot rule (minimum 24pt+ on 1080p).
- [ ] Gamepad navigation supports deterministic D-pad focus hopping with clear visual selection state.
- [ ] Button prompts dynamically update when switching between Xbox, PlayStation, and Keyboard.
- [ ] HUD text includes drop shadows or backings to remain readable against bright 3D backgrounds.

---

## Anti-Patterns

- **Microscopic PC Text on TV**: Forcing console players to squint at 11pt inventory text.
- **Mouse-Only Menu Design**: Requiring console players to drag an artificial virtual mouse cursor with an analog stick.
- **HUD Clutter**: Covering 60% of the game screen with static meters, blocking the player's view of the game world.
