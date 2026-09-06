---
name: unity-multiplayer
group: Unity
description: Build Netcode for GameObjects, Relay and Lobby, server authority and lag compensation. For exploit defense, see anti-cheat. Use when implementing Netcode for GameObjects or sync.
---

# unity-multiplayer

## Core Philosophy
Multiplayer game programming is an exercise in distributed systems operating under the constraints of physics and latency. You cannot trust the client; anything calculated on a player's machine—hit detection, movement speed, inventory transactions—will inevitably be hacked. High-performance Unity multiplayer requires a server-authoritative model with Netcode for GameObjects (NGO), Unity Relay & Lobby for NAT traversal, and client-side prediction with server reconciliation to mask network latency for players.

---

## 4-Step Unity Multiplayer Architecture

### Step 1: Topology & Transport (NGO + Relay + Lobby)
1. **Network Topology**:
   - **Dedicated Game Server (DGS)**: Absolute authority, zero host advantage, highest server cost. Best for competitive matchmaking.
   - **Listen Server / Host-Client**: One player acts as both host and server. Zero server cost, but host has 0ms latency advantage.
2. **Unity Transport & Relay**:
   - Use Unity Transport Package (UTP) combined with Unity Relay service to pierce corporate firewalls and symmetric NATs without requiring players to configure router port forwarding.
3. **Unity Lobby**:
   - Handles room matchmaking, player slot reservations, and custom lobby metadata before spinning up the network transport.

### Step 2: Server Authority & NetworkVariables
1. **`NetworkBehaviour` & Ownership**:
   - Code checks `IsServer`, `IsClient`, and `IsOwner`:
     ```csharp
     if (!IsOwner) return; // Only local player handles input
     ```
2. **`NetworkVariable<T>`**:
   - Replicates state from Server $\to$ Clients automatically:
     ```csharp
     public NetworkVariable<int> Health = new(
         100, 
         NetworkVariableReadPermission.Everyone, 
         NetworkVariableWritePermission.Server // Never allow client writes!
     );
     ```

### Step 3: Remote Procedure Calls (RPCs)
1. **ServerRpc**: Client calls, Server executes (e.g. *RequestFireWeaponServerRpc*).
2. **ClientRpc**: Server calls, all Clients execute (e.g. *PlayExplosionFxClientRpc*).
3. **RPC Rule of Thumb**:
   - Use `NetworkVariable` for persistent state (Health, Gold, Position).
   - Use `RPCs` for transient one-off events (Sound effects, particle spawns).

### Step 4: Lag Compensation & Client Prediction
1. **Client-Side Prediction**:
   - Client immediately applies local input and updates position locally without waiting for server confirmation round-trip.
2. **Server Reconciliation**:
   - Server validates movement:
     $$\Delta \vec{p} \le \vec{v}_{\text{max}} \times \Delta t + \epsilon$$
   - If client position drifts beyond error threshold $\epsilon$, server overwrites client position with authoritative coordinates, and client smoothly blends to the corrected position.
3. **Lag Compensated Hitbox History**:
   - Server rewinds player hitboxes back in time by the attacker's latency $T_{\text{rtt}} / 2$ to verify if the shot lined up when the client pulled the trigger.

---

## Deliverable Format: Server-Authoritative NGO Weapon Firing Pattern

```csharp
using Unity.Netcode;
using UnityEngine;

public class NetworkPlayerWeapon : NetworkBehaviour {
    [SerializeField] private GameObject bulletPrefab;
    [SerializeField] private Transform muzzleTransform;

    private void Update() {
        if (!IsOwner) return;
        if (Input.GetButtonDown("Fire1")) {
            // Request firing to the authoritative server
            FireWeaponServerRpc(muzzleTransform.position, muzzleTransform.forward);
        }
    }

    [ServerRpc]
    private void FireWeaponServerRpc(Vector3 origin, Vector3 direction) {
        // Server validation check
        if (Vector3.Distance(origin, muzzleTransform.position) > 1.5f) {
            Debug.LogWarning($"[Security] Player {OwnerClientId} origin spoofing detected!");
            return;
        }

        // Spawn networked bullet
        GameObject bullet = Instantiate(bulletPrefab, origin, Quaternion.LookRotation(direction));
        bullet.GetComponent<NetworkObject>().Spawn();
        
        // Notify all clients to play audio/vfx
        PlayMuzzleFlashClientRpc();
    }

    [ClientRpc]
    private void PlayMuzzleFlashClientRpc() {
        // Play local visual muzzle flash and sound effect
    }
}
```

---

## Worked Example: Resolving Rubberbanding in a Fast-Paced Arena Shooter

- **Problem**: In an action arena shooter built on NGO, players with 120ms ping experienced jarring stutter and teleportation (rubberbanding) whenever sprinting.
- **Root Cause**: The server was transmitting absolute position snapshots at 20Hz; client prediction was lacking, snapping the local player to historical server coordinates.
- **Solution**: Implemented tick-based input buffering and reconciliation. Client stores a history buffer of unconfirmed input ticks. When receiving a server correction, the client re-simulates all unacknowledged ticks from the corrected state.
- **Result**: Movement became completely smooth for players with up to 180ms latency.

---

## Verification Checklist

- [ ] All `NetworkVariable` write permissions set strictly to `Server`.
- [ ] Unity Relay service handles NAT punchthrough without manual port forwarding.
- [ ] Client movement inputs validated server-side against maximum acceleration limits.
- [ ] Weapon hit registration verified on server using historical hitbox rewind.
- [ ] Network objects cleanly despawned using `NetworkObject.Despawn()` on disconnect.

---

## Anti-Patterns

- **Client-Authoritative Damage**: Allowing clients to send `DealDamageServerRpc(targetId, 9999)` without server-side weapon range, line-of-sight, or ammo validation.
- **Over-Broadcasting RPCs Every Frame**: Sending continuous transform updates via RPCs instead of compressed `NetworkTransform` ticks.
- **Ignoring Network Disconnections**: Leaving orphaned player GameObjects in the scene when a client forcibly disconnects or loses connection.
