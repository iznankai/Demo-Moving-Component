
# MovingComponent | Plugin Documentation

**Version:** 1.0  
**Author:** Iznankai, Inc.  
**Category:** Other

---

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Components](#components)
5. [MovingComponent](#movingcomponent)
6. [Movement States (EMovingState)](#movement-states-emovingstate)
7. [FMovingTarget Structure](#fmovingtarget-structure)
8. [Properties and Parameters](#properties-and-parameters)
9. [Push Interaction](#push-interaction)
10. [API Reference](#api-reference)
11. [BitOperationsLibrary](#bitoperationslibrary)
12. [Usage Examples](#usage-examples)
    - [Configuring Push: Door Always Opens Away from the Player](#configuring-push-door-always-opens-away-from-the-player)

---

## Overview

**MovingComp** is an Unreal Engine plugin that provides a component for interpolating scene components between two transforms. It supports forward (Play) and reverse (Reverse) movement with configurable speed, collision handling, and an optional **Push Interaction** mode for doors and similar objects.

### Key Features

- Interpolation of one or more components between start and end positions
- Five states: Start, End, Play, Reverse, Paused
- Configurable movement speed in both directions
- Collision disabling during movement
- Dynamic navigation affect (NavMesh)
- **Push Interaction** - open/close based on which side the actor approaches from
- State change events and overlap detection
- Full Blueprint and C++ support

---

## Installation

1. Copy the "MovingComp" folder to your Unreal Engine project's "Plugins" directory.
2. Open the project - the plugin will be loaded automatically.
3. If needed, enable the plugin in **Edit ? Plugins** (category **Other**).

---

## Quick Start

1. Add the **Moving Component** to any Actor.
2. In the **Moving Targets** array, specify the component and set **Transform Start** and **Transform End**.
3. Call "SetMovingState(Play)" to start forward movement or "SetMovingState(Reverse)" for reverse movement.
4. Subscribe to the "OnChangeMovingState" delegate to react to state changes.

---

## Components

| Component | Description |
|-----------|-------------|
| **UMovingComponent** | Main movement interpolation component |
| **UBitOperationsLibrary** | Bit flag operations library |

---

## MovingComponent

"UMovingComponent" is an Actor Component that interpolates scene components between two transforms. It supports forward (Play) and reverse (Reverse) movement with configurable speed and collision handling.

### Hierarchy

```
UActorComponent
    ??? UMovingComponent
```

---

## Movement States (EMovingState)

| State | Value | Description |
|-------|-------|-------------|
| **None** | 0 | Hidden, internal use |
| **Start** | 1 | Component at start position (Alpha = 0) |
| **End** | 2 | Component at end position (Alpha = 1) |
| **Play** | 4 | Moving from Start to End |
| **Reverse** | 8 | Moving from End to Start |
| **Paused** | 16 | Movement paused between Start and End |

---

## FMovingTarget Structure

Defines a single movement target for interpolation.

| Field | Type | Description |
|-------|------|-------------|
| **Component** | FComponentReference | Reference to the scene component |
| **TransformStart** | FTransform | Transform at Alpha = 0 |
| **TransformEnd** | FTransform | Transform at Alpha = 1 |
| **TransformEndBack** | FTransform | Optional end transform when opening from the back side (Push Interaction) |

---

## Properties and Parameters

### Main

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| **Moving Targets** | TArray\<FMovingTarget\> | - | Array of targets for interpolation |
| **State** | EMovingState | Start | Current state |
| **Alpha** | float | 0.0 | Interpolation (0 = Start, 1 = End) |
| **Play Speed** | float | 1.0 | Movement speed Start ? End |
| **Reverse Speed** | float | 1.0 | Movement speed End ? Start |
| **Use Sweep** | bool | false | Use sweep when setting transform |
| **Teleport Type** | ETeleportType | TeleportPhysics | Teleport type when setting transform |
| **Tick Interval** | float | 0.0 | Transform update interval (0 = every tick) |
| **Notify All State Changes** | bool | false | Fire events on redundant transitions |
| **bDynamic Nav Affect** | bool | true | Dynamically change navigation affect |
| **bDisable Collision During Movement** | bool | true | Disable collisions during movement |

### Push Settings (when bEnablePushInteraction = true)

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| **bEnable Push Interaction** | bool | false | Enable Push Interaction mode |
| **Select Front Axis** | FVector | (0, 1, 0) | Axis defining the "front" side in local space |
| **Push Area Scale** | FVector | (1.1, 1.1, 1.1) | Overlap zone scale for push detection |
| **Stop Area Scale** | FVector | (1.2, 1.2, 1.2) | Overlap zone scale for stop detection |
| **bPush By Physics** | bool | false | Use physics (Reverse) instead of manual Alpha adjustment |
| **bUse Debug** | bool | false | Enable overlap debug visualization |
| **Disallow Push Objects** | TSet\<TSubclassOf\<AActor\>\> | - | Actor classes that cannot push |
| **Ignore Actors By Class** | TSet\<TSoftClassPtr\<AActor\>\> | - | Actor classes ignored during overlap |

---

## Push Interaction

**Push Interaction** mode allows opening and closing objects (doors, gates, etc.) based on which side the actor approaches from.

### Overlap Types (EAvoidingType)

| Type | Value | Description |
|------|-------|-------------|
| **Free** | 0 | No blocking actors |
| **Front** | 4 | Actors on the front side |
| **Back** | 8 | Actors on the back side |
| **Stack** | Front \| Back | Actors on both sides (movement blocked) |

### Logic

- **Move(Instigator, Location)** - opens toward the given Location (e.g., player position).
- **MoveFront(Instigator, bOpenSide)** - opens toward the specified side.
- **MoveBack(Instigator)** - closes (transitions to Reverse).
- With **Stack**, movement is not performed.
- With an actor on the opposite side: either Reverse (if "bPushByPhysics"), or manual Alpha decrease.

---

## API Reference

### State Control

| Function | Description |
|----------|-------------|
| "SetMovingState(EMovingState NewState, AActor* InInstigator)" | Set movement state |
| "SetAlpha(float NewAlpha)" | Set Alpha (0.0-1.0) |
| "UpdateTransform()" | Update transforms of all targets based on current Alpha |

### State Reading

| Function | Returns | Description |
|----------|---------|-------------|
| "GetAlpha()" | float | Current Alpha |
| "GetMovingState()" | EMovingState | Current state |
| "GetInstigator()" | AActor* | Actor that triggered the last state change |
| "IsInTransition()" | bool | true if Play or Reverse |
| "InEndOrPlayPosition()" | bool | true if Play or End |
| "GetPlaySpeed()" | float | Play speed |
| "GetReverseSpeed()" | float | Reverse speed |

### Push Interaction

| Function | Description |
|----------|-------------|
| "IsFront(const FVector& Location)" | true if Location is on the front side |
| "Move(AActor* InInstigator, const FVector& Location)" | Open toward Location |
| "MoveFront(AActor* InInstigator, bool bOpenSide)" | Open toward the specified side |
| "MoveBack(AActor* InInstigator)" | Close |
| "SetCustomSpeed(float InPlaySpeed, float InReverseSpeed)" | Set speeds |
| "AddIgnoreClassActor(TSoftClassPtr\<AActor\> Class)" | Add class to ignore list |
| "SetAvoidingActors(const TArray\<AActor*\>& NewActorsSet)" | Set list of actors to avoid |

### Delegates

| Delegate | Signature | Description |
|----------|------------|-------------|
| **OnChangeMovingState** | (EMovingState NewState, EMovingState PreviousState) | Fired when state changes |
| **OnHitActor** | (UMovingComponent*, AActor*, EAvoidingType) | Fired on overlap with actor (Push Interaction) |

---

## BitOperationsLibrary

Blueprint and C++ library for bit flag operations.

| Function | Description |
|----------|-------------|
| "HasFlag(Compare, Flag)" | true if Flag is set in Compare |
| "ContainsFlag(Compare, Flag)" | true if Compare contains any of Flag's bits |
| "AddFlag(Compare, Flag)" | Add Flag to Compare |
| "RemoveFlag(Compare, Flag)" | Remove Flag from Compare |
| "ClearFlag(Compare)" | Clear all bits in Compare |

---

## Usage Examples

### Basic Door Movement

1. Add Moving Component to the door Actor.
2. In Moving Targets, specify the door mesh and set Transform Start (closed) and Transform End (open).
3. Call "SetMovingState(Play)" to open, "SetMovingState(Reverse)" to close.

### Door with Push Interaction

1. Enable **bEnable Push Interaction**.
2. Configure **Select Front Axis** (direction of the "front" side).
3. On interaction, call "Move(Player, Player->GetActorLocation())" - the door will open toward the player.
4. Subscribe to "OnHitActor" to react to overlaps with actors.

### Configuring Push: Door Always Opens Away from the Player

To make the door always open **away** from the player (in the direction opposite to their position):

1. **Select Front Axis** - set the axis pointing outward from the door hinge toward one side (e.g. (0, 1, 0) for Y). The front side is determined so that the player is "in front" when the dot product of this axis with the direction to the player is negative.

2. **Transform Start** - closed door position.

3. **Transform End** - end position when the player pushes from the **front** side (door swings away from them, toward the back).

4. **Transform End Back** - end position when the player pushes from the **back** side (door swings away from them, toward the front). If left as Identity, Transform End is used - the door will open to the same position from both sides.

5. On interaction, call "Move(Player, Player->GetActorLocation())" - the component will automatically detect which side the player is on and choose the correct end transform.

6. **Push Area Scale** and **Stop Area Scale** - if needed, increase them (e.g. to 1.2-1.5) so the overlap zone better covers the space around the door.

### Blueprint: Reacting to State Change

```
Event OnChangeMovingState (NewState, PreviousState)
    if NewState == End ? play close sound
    if NewState == Start ? play open sound
```

---
