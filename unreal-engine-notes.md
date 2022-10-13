# Unreal Engine

## Table of Contents
- [AI](#ai)
- [Animation](#animation)
  - [Animation Blueprint](#animation-blueprint)
  - [AnimGraph](#animgraph)
  - [Event Graph](#abp-event-graph)
  - [Animation Montage](#animation-montage)
  - [AnimNotifies](#animnotifies)
- [Light](#light)
  - [Sky Light](#sky-light)
  - [Ambient Light](#ambient-light)
- [Physics](#physics)
  - [Collision](#collision)

# AI

### AI Controller

 - See [AI Controller Object](./unreal-objects.md#ai-controllers)

### Navigation Mesh

- AI Need a navigation messh to show where they can and cannot navigate

- Found in **Volume > Nav Mesh Bounds Volume**

- **Tip:** [CharacterMovementComponent](./unreal-objects.md#UCharacterMovementComponent) `StopMovement()` + `DisableMovement()` will stop current movement immediately, and disable acceleration

# Animation

### Skeletal Meshes

- Can be animated.

- **Sockets** used to attach meshes to skeletal meshes

## Animation Blueprint

- Controls the animation of a skeletal mesh

- **Note:** Remember to assign the **AnimationBlueprint** to the Character BP

- Has both an **Event Graph** and **Anim Graph**

- Store external variables locally in the AnimationBlueprint Event Graph, then access them in the **AnimGraph**

## AnimGraph

Generally speaking, to set and transition animations you can use blend or a state machine

### Blend Poses

- [Documentation](https://docs.unrealengine.com/4.27/en-US/AnimatingObjects/SkeletalMeshAnimation/NodeReference/Blend/)

- **Blend Poses By Bool** a node that performs a time-based blend between two poses using a Boolean value as the key.

### StateMachine

- [Documentation](https://docs.unrealengine.com/5.0/en-US/state-machines-in-unreal-engine/)

- **Transition Rules** manage transition between states
  - Example Usage Flow: **ABP > AnimGraph > StateMachine**
    - **Add State > Assign Animation**
      - **Note:** Can set Loop behavior
    - Link States with Transition rules

- **Blendspace** blends between animation
  - **Note:** in a **Statemachine State** you want to use a **blendspace** to blend between animation
  - Example Usage Flow: **ABP > AnimGraph > StateMachine > State > Blendspace > Drag and drop animation to graph point**

## ABP Event Graph
  - Can access the AnimationBP inside a Pawn’s EventGraph *via the Pawn’s associated Mesh*
  - **Tip:** In the event graph can get AnimationBlueprint owner pawn and cast to class to access info (ie velocity)

## Animation Montage

You can use Animation Montages (Montages) to combine several Animation Sequences into a single asset and control playback with Blueprints. You can also use Animation Montages to replicate Root Motion animation in network games. 

- [Documentation](https://docs.unrealengine.com/5.0/en-US/animation-montage-in-unreal-engine/)

- Can trigguer animations from inside BP Event Graphs via **Play Montage** node

- Can create by assembling Animations in in content folder

- Animation Montages are assigned **slots** to organize, accessable from inside the **AnimBP**. The slot can be hooked up to an AnimBP AnimGraph between the state machine and the output pose. This will allow the AnimBP to play the montage assocated with the slot

- Can add [**AnimNotifies**](#animnotifies) **Trigger** in the Animation Montage

## AnimNotifies

- Animation Notifications
- Used to spawn sound effects, particle effects and execute code
- Can add listeners inside an AnimationBlueprint Event Graph

# Light

- Three types of rendering: **Static, Stationary,** and **Movable**
  
  - Static and Stationary are Pre-rendered.
  
  - Moveable is rendered dynamically

- **Lumen** - UE5 light renderer
  
  - Works best with **Moveable** lights
  
  - Buggy with Material "Pixel Depth Offset"

- **AutoExposure** (aka Eye Adaptation)
  
  - Post processing effect of the eye adjusting to ambient light (ie Bethesda "Step out moment").
  
  - Available in **Project Settings**
  
  - Alternatively can add a [PostProcessingVolume](./unreal-objects.md#postprocessingvolume), then go to **Exposure > Metering** and switch from **AutoExposureHistogram** to **AutoExposureBasic**.

### Sky Light

- Calculates light from the skybox

- Alternatively can set a default cube map to apply ambient light

### Ambient Light

- Can connect Sky Light and Directional Light so that the Sky light properties accurately reflect the angle of the directional light.
  
  - The skybox will move the sun to where the Direction source is

# Physics

## Collision
