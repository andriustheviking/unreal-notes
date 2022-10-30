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
- [Material](#material)
- [Particles](#particles)
- [Sounds](#sounds)

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

- Meshes that use the same skeleton can use the same animations.

- **Sockets** used to attach meshes to skeletal meshes

## Animation Blueprint

- Controls the animation to a specific skeletal mesh. The skeleton is selected when the ABP is created

- **Note:** Remember to assign the **AnimationBlueprint** to the Character BP

- Has both an **Event Graph** and **Anim Graph**

- Store external variables locally in the AnimationBlueprint Event Graph, then access them in the **AnimGraph**

## AnimGraph

Generally speaking, to set and transition animations you can use blend or a state machine

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

    - 

### StateMachine

- [Documentation](https://docs.unrealengine.com/5.0/en-US/state-machines-in-unreal-engine/)

- **Transition Rules** manage transition between states
  - Example Usage Flow: **ABP > AnimGraph > StateMachine**
    - **Add State > Assign Animation**
      - **Note:** Can set Loop behavior
    - Link States with Transition rules

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

- `WakeAllRigidBodies()` - Physics enabled components that don’t move in awhile will need to “woken up” to re-activate physics

- [UPhysicsHandleComponent](./unreal-objects.md#UPhysicsHandleComponent) - Add to a blueprint (or actor) to add interface for grabbability, rotation etc

## Collision

- Unreal only considers the root component for collisions.

- To enable collision set the collision mesh to Blueprint’s **DefaultSceneRoot**, and set the motion event (ie [AddActorLocalOffset](./unreal-objects#actor-methods)) to **Sweep**

- **Note:** An actor that doesn’t have their collision as the root can still block other actors. But, if you move the actor, it will not collide with anything

- There are three types of Collision Responses:
  - **Ignore**
  - **Overlap** (Make sure an actor has **Generate Overlap Events** set)
  - **Block**

- **Teleport** - setting that maintains velociy of object, preventing random jumps in collision

- **Continuous Collision Detection (CCD)** - setting that allows more accurate collision, but more computationally expensive

- **Collision Mesh** 
  - Autogenerated when the mesh is imported (if enabled on import). 
  - Can replace with custom collision mesh.

- **Collision Component**
  - You can add them through the Components panel. 
  - `UShapeComponents` used for simple collision.
  - These come in three shapes: `UBoxComponent`, `UCapsuleComponent` and `USphereComponent`. 

- **[FComponentHitSignature OnComponentHit](./unreal-objects.md#uprimitivecomponent-methods)**

# Material

- Documentation
  - [Physically Based Materials in Unreal Engine](https://docs.unrealengine.com/5.0/en-US/physically-based-materials-in-unreal-engine/)
  - [Content Examples Sample Project for Unreal Engine](https://docs.unrealengine.com/5.0/en-US/content-examples-sample-project-for-unreal-engine/)
  - [Instanced Materials](https://docs.unrealengine.com/4.27/en-US/RenderingAndGraphics/Materials/MaterialInstances/)

- Materials can be made from Textures

- **Material Graph** sets and controls the properties of the material
		- Nodes connect via pins input/output to modify the material
		- Result Node is the final material result
		- Drag textures to the graph and connect them to the appropriate node property (result node, or other)    

- Materials contains Elements
  - Useful basic elements:
		- **Base Color**: The color or texture of a surface. Used to add detail and color variations.
		- **Metallic**: How “metal-like” a surface is. Generally, a pure metal will have the maximum Metallic value whereas fabric will have a value of zero.
		- **Specular**: Controls the shininess of nonmetallic surfaces. For example, ceramic would have a high Specular value but clay would not.
		- **Roughness**: A surface with maximum roughness will not have any shininess. It’s used for surfaces such as rock and wood.
		- **Emissive**: glowiness

### DynamicMaterialInstance

You can only create dynamic material instances during gameplay. You can use Blueprints (or C++) to do this. So the material is spawned during gameplay by a triggering event

# Particles

- Can be assigned to a socket on a skeletal mesh
- Can be Activated / Deactivated
- See [SpawnEmitterAtLocation](./unreal-objects.md#SpawnEmitterAtLocation)
- See [UParticleSystem](./unreal-objects.md#UParticleSystem) 
- See [UParticleSystemComponent](./unreal-objects.md#UParticleSystemComponent)

# Sounds

- **Sound Cues** can edit and modularize soundsfiles together
- **Spawn Sound** will spawn an audio component object instance
	- This can be at a Location, Attached to object, or 2D (ie non-spacialized)
	- **Stop** will stop the sound cue
	- **AutoDestroy** the sound component instance to remove it from the game
- See [Sound](./unreal-objects.md#sound) for implelmentation details