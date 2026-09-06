---
name: unity-architecture
group: Unity
description: >-
  Structure Unity projects with ScriptableObjects, decoupled systems and single-responsibility
  components. Use when structuring Unity projects, ScriptableObjects, or DOTS/ECS.
---

# unity-architecture

## Core Philosophy
Monolithic `MonoBehaviour` architectures turn Unity projects into unmaintainable spiderwebs. When every script references every other script via `GetComponent`, `FindObjectOfType`, or static singletons, changing a player's health logic breaks the UI, sound manager, and save system simultaneously. Scalable Unity architecture relies on **ScriptableObject-driven architecture** (Ryan Hipple pattern), event-driven decoupling, Single Responsibility components, and data-oriented design (DOTS/ECS) for performance-critical systems.

---

## 4-Step Unity Architecture Patterns

### Step 1: ScriptableObject Architecture (Modular Data & Events)
1. **ScriptableObject Variables**:
   - Store shared game state (Player Health, Ammo, Score) in ScriptableObject assets instead of static singletons:
     ```csharp
     [CreateAssetMenu(menuName = "Variables/FloatVariable")]
     public class FloatVariable : ScriptableObject {
         public float Value;
     }
     ```
2. **ScriptableObject Game Events**:
   - Broadcast events without direct coupling between caller and listener:
     ```csharp
     [CreateAssetMenu(menuName = "Events/GameEvent")]
     public class GameEvent : ScriptableObject {
         private readonly List<GameEventListener> listeners = new();

         public void Raise() {
             for (int i = listeners.Count - 1; i >= 0; i--)
                 listeners[i].OnEventRaised();
         }
         public void Register(GameEventListener l) => listeners.Add(l);
         public void Unregister(GameEventListener l) => listeners.Remove(l);
     }
     ```

### Step 2: Decoupled Component Architecture
1. **Single Responsibility Components**:
   - Break mega-scripts like `PlayerController.cs` (3,000 lines) into isolated single-purpose components:
     - `PlayerInputReader.cs` (Translates raw inputs to movement vectors).
     - `PlayerMotor.cs` (Executes CharacterController physics).
     - `PlayerHealth.cs` (Manages damage calculations and invokes events).
     - `PlayerAudioFeedback.cs` (Listens to health events and plays footstep/hit clips).
2. **Interface Segregation**:
   - Use interfaces for subsystem interactions (e.g. `IDamageable`, `IInteractable`).

### Step 3: Runtime Sets for Dynamic Entity Tracking
1. **Replacing `FindObjectsOfType`**:
   - Create a `RuntimeSet<T>` ScriptableObject. When an enemy spawns, it adds itself to `EnemyRuntimeSet`; on destroy, it removes itself.
   - Systems query the set in $O(1)$ time with zero GC allocation or scene tree traversal.

### Step 4: DOTS / ECS for Scale (When to Reach for Data-Oriented Tech)
1. **Object-Oriented vs Data-Oriented (DOTS)**:
   - Use standard `MonoBehaviour` + `ScriptableObjects` for high-level gameplay, menus, and quests.
   - Use Unity DOTS (Entities, IJobEntity, Burst Compiler) when simulating $> 5,000$ concurrent active entities (e.g. bullet hell projectiles, swarm crowds, voxel terrain).

---

## Deliverable Format: Decoupled ScriptableObject Event System

```csharp
// GameEventListener.cs: Attach to any GameObject that reacts to an event
using UnityEngine;
using UnityEngine.Events;

public class GameEventListener : MonoBehaviour {
    [SerializeField] private GameEvent gameEvent;
    [SerializeField] private UnityEvent response;

    private void OnEnable() => gameEvent.Register(this);
    private void OnDisable() => gameEvent.Unregister(this);
    public void OnEventRaised() => response.Invoke();
}
```

---

## Worked Example: Refactoring a Spaghetti Boss Fight

- **Problem**: In a boss encounter, `BossAI.cs` held direct references to `PlayerController`, `UIManager`, `CameraShake`, `AudioManager`, and `SaveSystem`. Updating the UI caused null reference exceptions in Boss AI during scene transitions.
- **Refactoring**:
  1. Replaced direct references with 2 ScriptableObject events: `OnBossPhaseChanged` and `OnBossDied`.
  2. UI, Camera, Audio, and Save scripts registered as independent listeners via `GameEventListener`.
- **Result**: `BossAI.cs` line count reduced from 1,420 lines to 310 lines. Zero scene reload reference bugs.

---

## Verification Checklist

- [ ] Zero usage of `FindObjectOfType()` or string-based `SendMessage()` in runtime loops.
- [ ] Shared state stored in `ScriptableObject` assets rather than mutable static singletons.
- [ ] Game events broadcast via ScriptableObject events or C# delegates.
- [ ] Entity discovery utilizes `RuntimeSets` rather than heavy scene scans.
- [ ] Heavy simulations (> 5,000 entities) isolated into DOTS / Burst-compiled jobs.

---

## Anti-Patterns

- **God Singletons**: Creating a `GameManager.cs` singleton that controls player input, UI, high scores, audio, and network sockets in 4,000 lines.
- **Tight Coupling via `GetComponent` in `Update()`**: Calling `GetComponent<Animator>()` every frame inside `Update()`.
- **Scene-Bound State**: Storing high scores and inventory state on scene objects, causing data loss upon `SceneManager.LoadScene()`.
