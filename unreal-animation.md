# Unreal Animation

## Table of Contents

- [Skeletal Mesh](#skeletal-meshes)
- [Animation Blueprint](#animation-blueprint)
  - [Event Graph](#abp-event-graph)
  - [Animation Montage](#animation-montage)
  - [AnimNotifies](#animnotifies)
  - [AnimGraph](#animgraph)
---

# Skeletal Meshes

- Can be animated.

- Meshes that use the same skeleton can use the same animations.

- **Sockets** used to attach meshes to skeletal meshes

- **Tip:** Can hide mesh subcomponents by hiding the bone its attached to.

# Animation Blueprint

- Controls the animation to a specific skeletal mesh. The skeleton is selected when the ABP is created

- **Note:** Remember to assign the **AnimationBlueprint** to the Character BP

- Has both an **Event Graph** and **Anim Graph**

- Store external variables locally in the AnimationBlueprint Event Graph, then access them in the **AnimGraph**


## AnimNotifies

- Animation Notifications
- Used to spawn sound effects, particle effects and execute code
- Can add listeners inside an AnimationBlueprint Event Graph

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

## Additive Animation

- Can be added on top of other animations. **Example:** Aim Offset Animations

- Takes a "Base Pose" and modifies it from inputs

## AnimGraph

Generally speaking, to set and transition animations you can use blend or a state machine

Example Usage Flow: **ABP > AnimGraph > StateMachine**

### Blend Poses

- [Documentation](https://docs.unrealengine.com/4.27/en-US/AnimatingObjects/SkeletalMeshAnimation/NodeReference/Blend/)

- There are many different kinds of blends. These blend animations based off of a value

- **Blend** standard blend uses an alpha channel

- **Blend Poses By Bool** a node that performs a time-based blend between two poses using a Boolean value as the key.

### Blendspace

- Blends between multiple animations via 2D graph

- Great for locomotion. 

- Can reference Blendspaces directly in AnimGraph, or using StateMachine

- **Note:** in a **Statemachine State** you want to use a **blendspace** to blend between animation

- **Tips:**
  
  - Enable labels in blendspace to see the animation labels
  
  - Tap Ctrl on keyboard to move preview point under mouse cursor

- Example Usage Flow: **ABP > AnimGraph > StateMachine > State > Blendspace > Drag and drop animation to graph point**

#### Character Movement Animation Using Blendspace

- In blendspace, use Angle for X-axis (-180, 180)  and Speed for Y-axis, as opposed to Forward / Right. This will allow better transition for all angles of movement.

- Getting actor relative velocity direction:
  
  - **Transform** methods convert local space to global space, while **InverseTransform** global to local. So to get the velocity of an object *relative to itself,* use an **InverseTransform** of the global velocity, passing in the Actor Transform, to get the velocity of the actor relative its own transform.
  
  - With the actor relative transform, convert the vector to a Rotator and get the Yaw

- Match animation speed to global velocity to prevent sliding:
  
  - Can calculate foot speed of animation and then set the ranges on the BS graph

### StateMachine

- [Documentation](https://docs.unrealengine.com/5.0/en-US/state-machines-in-unreal-engine/)

- StateMachines are primarily compose of **States** and **Transisions**

#### States
  
  - Output animations
  - Add animations in a state
  - **Note:** Can set Loop behavior for animation

#### Transitions

  - Determine the transition between states, and have their own logic

  - **Transition Rules:** 

    - Can add rule logic in the Transition Graph

    - **Automatic Rule Base on Sequence Player in State** 
      - Can check this directly on the Transition Detail.
      - Automatically transitions one state to the next based on the animation progress.
      - Can modify this transition based of the **Blend** property in Details
