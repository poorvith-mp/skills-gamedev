---
name: unreal-systems
group: Unreal
description: >-
  Work the C++/Blueprint continuum with Nanite, Lumen and the Gameplay Ability System. Use when
  developing Unreal Engine gameplay in C++, Blueprints, or GAS.
---

# unreal-systems

## Core Philosophy
Modern Unreal Engine 5 development is anchored by three technological pillars: **Nanite** (virtualized micropolygon geometry), **Lumen** (real-time dynamic global illumination and reflections), and the **Gameplay Ability System (GAS)**. Building scalable Unreal systems requires navigating the C++ vs Blueprint continuum: architecting core performance-critical mechanics and foundational types in clean C++, while exposing flexible tuning parameters, abilities, and cosmetic triggers to designers in Blueprints.

---

## 4-Step Unreal Engine 5 Systems Architecture

### Step 1: The C++ vs Blueprint Continuum
1. **The Golden Rule**:
   - **C++**: Math, algorithmic logic, data structures, multiplayer networking, file I/O, base actor classes, Gameplay Ability tasks.
   - **Blueprints**: Visual assembly, animation state machine hooks, cosmetic audio/particle binding, rapid iteration tuning.
2. **No Pure Blueprint Actors**:
   - Every production Blueprint actor must derive from a custom C++ base class (`AMyGameCharacter -> BP_MyGameCharacter`). Never derive directly from native engine classes in Blueprint.

### Step 2: Gameplay Ability System (GAS) Architecture
1. **The Core Components of GAS**:
   - **`UAbilitySystemComponent` (ASC)**: The brain component attached to any character that can use abilities.
   - **`UAttributeSet`**: Manages numerical attributes (Health, Stamina, Armor) with clamping and modification hooks:
     - `PreAttributeChange()` and `PostGameplayEffectExecute()`.
   - **Gameplay Tags (`FGameplayTag`)**: Hierarchical string-like identifiers for state and categorization (e.g. `State.Debuff.Stunned`, `Ability.Melee.HeavyAttack`).
   - **Gameplay Effects (GE)**: Data-driven modifications to attributes (Instant damage, Duration buffs, Periodic poison).
   - **Gameplay Abilities (GA)**: Scripted actions executed via Ability Tasks (e.g. *PlayMontageAndWait*).

### Step 3: Nanite Virtualized Geometry Rules
1. **Nanite Asset Standards**:
   - Import high-poly ZBrush sculpts (1M - 10M triangles) directly without manual LOD generation.
   - *Nanite Constraints*:
     - Nanite does NOT support non-rigid deforming skeletal meshes (characters) in earlier UE5 versions; reserve for static meshes and rigid foliage.
     - Avoid complex masked/translucent materials on Nanite geometry (forces software rasterizer fallback).

### Step 4: Lumen Global Illumination Optimization
1. **Software vs Hardware Ray Tracing**:
   - **Software Ray Tracing (Default)**: Uses Mesh Distance Fields and Global Distance Fields. Highly performant across consoles.
   - **Hardware Ray Tracing (HWRT)**: Requires DX12 and dedicated ray-tracing GPU hardware.
2. **Mesh Distance Field Hygiene**:
   - Ensure all static meshes have manifold, two-sided geometry without open backfaces to prevent light leaking through walls.

---

## Deliverable Format: Production GAS Attribute Set (C++)

```cpp
// AttributeSetBase.h
#pragma once
#include "CoreMinimal.h"
#include "AttributeSet.h"
#include "AbilitySystemComponent.h"
#include "AttributeSetBase.generated.h"

#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
    GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
    GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
    GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)

UCLASS()
class UAttributeSetBase : public UAttributeSet {
    GENERATED_BODY()

public:
    UAttributeSetBase();

    UPROPERTY(BlueprintReadOnly, Category = "Attributes")
    FGameplayAttributeData Health;
    ATTRIBUTE_ACCESSORS(UAttributeSetBase, Health)

    UPROPERTY(BlueprintReadOnly, Category = "Attributes")
    FGameplayAttributeData MaxHealth;
    ATTRIBUTE_ACCESSORS(UAttributeSetBase, MaxHealth)

    virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) override;
    virtual void PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data) override;
};
```

---

## Worked Example: Eliminating 14ms Hitching via C++ GAS Migration

- **Problem**: In a 4-player cooperative action RPG, casting a meteor storm ability caused a severe 14ms frame drop on all clients.
- **Diagnosis**: Profiling in Unreal Insights revealed that the meteor ability was authored entirely in Blueprint, spawning 80 individual Blueprint actors, each executing tick functions and heavy cast operations (`Cast<ABP_Enemy>`).
- **Optimization**:
  1. Migrated the meteor storm into a C++ `UGameplayAbility` using an asynchronous `UAbilityTask`.
  2. Replaced Actor spawning with batched Gameplay Effects targeting enemy ASCs directly via Gameplay Tags.
- **Outcome**: Ability activation frame time dropped from 14ms to 0.18ms with zero hitching.

---

## Verification Checklist

- [ ] All gameplay classes inherit from custom C++ base classes, not raw engine types.
- [ ] Attributes use the `ATTRIBUTE_ACCESSORS` macro and clamp values safely.
- [ ] State transitions orchestrated via hierarchical `GameplayTags`.
- [ ] Nanite enabled on high-poly static environment meshes.
- [ ] Mesh Distance Fields generate cleanly without light bleed in Lumen visualization views.

---

## Anti-Patterns

- **Casting in High-Frequency Blueprint Ticks**: Using `Cast To BP_Character` inside `Event Tick`, destroying CPU cache locality.
- **Hard Object References in Blueprints**: Directly referencing heavy textures or audio in Blueprints, causing the entire asset tree to load into RAM upon opening the level.
- **Overriding Health via Setters Instead of Gameplay Effects**: Directly setting character health values without routing through the Ability System Component, bypassing buffs, shields, and damage mitigation rules.
