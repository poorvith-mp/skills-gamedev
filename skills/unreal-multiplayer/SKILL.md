---
name: unreal-multiplayer
last_reviewed: 2026-09-06
group: Unreal
description: >-
  Build Actor replication, GameMode/GameState architecture and server-authoritative gameplay. Use
  when building Unreal networking, actor replication, or RPCs.
---

# unreal-multiplayer

## Core Philosophy
Unreal Engine was built from its inception in 1998 as a multiplayer tournament engine. Networking in Unreal is deeply opinionated: it centers around a strict **Server-Authoritative Client-Server architecture** orchestrated by the GameMode, GameState, PlayerState, and PlayerController classes. The server runs the definitive physical simulation; clients are visual and audio playback terminals that send player intent. Master Unreal multiplayer by understanding the Actor Replication lifecycle, property replication with `DOREPLIFETIME`, and the strict division between Multicast, Client, and Server RPCs.

---

## 4-Step Unreal Multiplayer Architecture

### Step 1: The Core Gameplay Framework Classes
1. **GameMode (Server Only)**:
   - Exists exclusively on the authoritative server. Defines match rules, win conditions, player spawning (`SpawnDefaultPawnAtTransform`), and score tracking. Never replicated to clients.
2. **GameState (Server $\to$ All Clients)**:
   - Replicated to all machines. Tracks match state (Round time, Team scores, Match phase: *WaitingToStart, InProgress, PostMatch*).
3. **PlayerState (Server $\to$ All Clients)**:
   - Replicated to all machines. Tracks individual player attributes (Player Name, Ping, Kills, Current Score).
4. **PlayerController (Server $\leftrightarrow$ Owning Client)**:
   - Exists on the server and the *specific owning client*. Handles input translation, UI HUD management, and secure Server RPC communication.

### Step 2: Actor & Property Replication (`DOREPLIFETIME`)
1. **Actor Replication Setup**:
   - `bReplicates = true` in constructor.
   - `SetReplicateMovement(true)` for physical actors.
2. **Replicated Properties (`ReplicatedUsing`)**:
   - Replicate variables using `UPROPERTY(ReplicatedUsing = OnRep_Health)`.
   - Implement `GetLifetimeReplicatedProps`:
     ```cpp
     void ACharacterBase::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const {
         Super::GetLifetimeReplicatedProps(OutLifetimeProps);
         DOREPLIFETIME(ACharacterBase, Health);
         DOREPLIFETIME_CONDITION(ACharacterBase, SecretInventory, COND_OwnerOnly);
     }
     ```
3. **RepNotify Functions**:
   - `OnRep_Health()` executes automatically on clients whenever the server updates `Health`.

### Step 3: Remote Procedure Call (RPC) Classification
1. **Server RPC (`Server, Reliable, WithValidation`)**:
   - Called by Owning Client $\to$ Executed on Server.
   - Must implement `_Validate` (Security gate) and `_Implementation`:
     ```cpp
     bool AWeapon::ServerFire_Validate(FVector AimDir) { return AimDir.IsNormalized(); }
     void AWeapon::ServerFire_Implementation(FVector AimDir) { /* Authoritative Fire */ }
     ```
2. **Client RPC (`Client, Reliable`)**:
   - Called by Server $\to$ Executed on Owning Client only (e.g. *ClientShowHitMarker*).
3. **NetMulticast RPC (`NetMulticast, Unreliable`)**:
   - Called by Server $\to$ Executed on Server and *all* simulated clients (e.g. *MulticastPlayExplosionFX*).

### Step 4: Network Relevancy & Dormancy Optimization
1. **Network Relevancy (`IsNetRelevantFor`)**:
   - Actors beyond view distance or hidden behind terrain are not replicated to distant clients, conserving network bandwidth.
2. **Net Dormancy (`DORM_DormantAll`)**:
   - Static or rarely changing world actors (chests, switches) are set dormant until an event wakes them up, saving server tick budget.

---

## Deliverable Format: Authoritative Server-Validated Weapon Firing (C++)

```cpp
// Weapon.h
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Weapon.generated.h"

UCLASS()
class AWeapon : public AActor {
    GENERATED_BODY()

public:
    AWeapon();

    UPROPERTY(ReplicatedUsing = OnRep_AmmoCount, BlueprintReadOnly)
    int32 AmmoCount;

    UFUNCTION()
    void OnRep_AmmoCount();

    UFUNCTION(Server, Reliable, WithValidation)
    void ServerFire(FVector_NetQuantize TraceStart, FVector_NetQuantizeNormal AimDir);

    UFUNCTION(NetMulticast, Unreliable)
    void MulticastSpawnMuzzleFX();

protected:
    virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;
};
```

---

## Worked Example: Mitigating Dedicated Server Bandwidth Saturation

- **Context**: A 32-player Unreal shooter was experiencing severe packet drops and server tick degradation down to 14 Hz during firefights.
- **Diagnosis**: Using `NetProfile` and `Network Insights`, engineers discovered weapon firing was using `Reliable` NetMulticast RPCs and replicating unquantized 64-bit vector coordinates 30 times per second.
- **Optimization**:
  1. Converted `FVector` to `FVector_NetQuantize` (16-bit integer compression).
  2. Changed cosmetic FX RPCs from `Reliable` to `Unreliable`.
  3. Activated Net Dormancy on 800 static interactive loot containers in the map.
- **Outcome**: Server bandwidth per client decreased by 68%; server tick rate stabilized at 60 Hz.

---

## Verification Checklist

- [ ] GameMode contains zero client-side logic and is never accessed from client HUDs.
- [ ] All Server RPCs implement `_Validate()` security checks to prevent packet injection.
- [ ] `DOREPLIFETIME_CONDITION` used to prevent sending private data (e.g. inventory) to non-owners.
- [ ] Cosmetic effects transmitted via `Unreliable` Multicast RPCs.
- [ ] Net Relevancy distance and Net Dormancy configured for large maps.

---

## Anti-Patterns

- **Calling Server RPCs on Un-Owned Actors**: Invoking a Server RPC from a client on an Actor that is not possessed or owned by that client's PlayerController (fails silently).
- **Using Reliable RPCs for Frequent Ticks**: Marking high-frequency weapon fire or footstep RPCs as `Reliable`, causing TCP-style packet queues when latency spikes.
- **Accessing `AGameMode` on Clients**: Writing `GetWorld()->GetAuthGameMode()` inside client UI widgets, which returns `nullptr` on non-server machines.
