# Unreal Engine

## Table of Contents

- [Overview](#unreal-overview)
- [AI](#ai)
- [Animation](#animation)
- [Input](#input)
- [Light](#light)
  - [Sky Light](#sky-light)
  - [Ambient Light](#ambient-light)
- [Physics](#physics)
  - [Collision](#collision)
- [Material](#material)
- [Particles](#particles)
- [Sound](#sound)

# Unreal Overview

### Play start order of operations:

1. Unreal Loads [Map](./unreal-objects.md#map)
2. **Map** specifies a [GameMode](./unreal-objects.md#gamemode)
3. [PlayerController](./unreal-objects.md#aplayercontroller) joins the **Map**
4. **PlayerController** asks **GameMode** to spawn a [Pawn](./unreal-objects.md#apawn--aactor)
5. The **Pawn** is linked to the **PlayerController**

# AI

- See [AI Notes](./unreal-ai.md)

### Navigation Mesh

- AI Need a navigation messh to show where they can and cannot navigate

- Found in **Volume > Nav Mesh Bounds Volume**

- **Tip:** [CharacterMovementComponent](./unreal-objects.md#UCharacterMovementComponent) `StopMovement()` + `DisableMovement()` will stop current movement immediately, and disable acceleration

# Animation

- See [Animation Notes](./unreal-animation.md)

# Input

## [Enhanced Input Actions](https://docs.unrealengine.com/5.1/en-US/enhanced-input-in-unreal-engine/)

`#include "EnhancedInputComponent.h"`

**NOTE:** Must include the `EnhancedInput` module in your build.cs file

Enhanced Input Operates by creating input Action classes, as opposed to using strings to map inputThr

### InputActions

- Defines the Action type label *("Walk", "Run", "Heal", "Reload")*, and the Value type *(BOOL, Float, 2DVector, 3DVector)*

- Maps Triggers and Modifiers to it

- Input Actions are binded via registering Listener object: 

Header:
```cpp
  UFUNCTION() //Not sure needs to be a UFUNCTION ?
  void UpdateThrottle(const struct FInputActionValue& Value);

  UPROPERTY(BlueprintReadWrite, EditDefaultsOnly)
  TObjectPtr<class UInputAction> ThrottleAction;
```

Implementation:
```cpp
void AGoKart::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
  Super::SetupPlayerInputComponent(PlayerInputComponent);
  if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
  {
    // You can bind to any of the trigger events here by changing the "ETriggerEvent" enum value
    if (ThrottleAction)
    {
      EnhancedInputComponent->BindAction(ThrottleAction, ETriggerEvent::Ongoing, this, &AGoKart::UpdateThrottle);
}}}
```

### InputMappingContext

- Implements the actual KeyBindings to InputActions
- `InputMappingContexts` can be created for different gameplay contexts (ie walking vs swimming)

- Add the mapping context to the player's enhanced input subsystem:

```cpp
  UPROPERTY(EditDefaultsOnly, Category = "Components")
  TObjectPtr<class UInputMappingContext> MappingContext;
```
```cpp
  if (APlayerController* PC = Cast<APlayerController>(GetController()))
  {
    if (UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PC->GetLocalPlayer()))
    {
      Subsystem->ClearAllMappings();
      Subsystem->AddMappingContext(Mapping, 0);
    }
  }
```

### InputModifiers

- Input Modifiers are pre-processors that alter the raw input values that UE receives before sending them on to Input Triggers.
- ie changing the order of axes, implementing "dead zones", and converting axial input to world space. 

### Input Triggers

- Set conditions to trigger inputs, like holding a certain number of times, or "chorded" input patterns

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

### CollisionParams

See [`FCollisionQueryParams`](./unreal-objects.md#FCollisionQueryParams)

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

# Sound

- See Also [Sounds](./unreal-objects.md#sounds) for implelmentation details

## Definitions:
  - **Attenuation** - Closer sounds appear louder, vice-versa
  - **Spatialization** - L/R balance relative to user  

## Ambient Sound Actor

Will Play sound everywhere in a level. Ideal for background game music

## Sound Cues

Can edit and modularize soundsfiles together
  - Can Set to Solo or Mute during testing to isolate sound
  - Cues can be set as [USoundBase](./unreal-objects.md#USoundBase)
  - Use the editor to mix different Wave paterns via nodes
  - **Tip:** Select sounds from the Content Browser, and right click in the Editor to auto configure the selected group.
  - **Nodes:**
    - Modulator - Varies the pitch and volume to within a selected range
    - Random Node - randomizes execution
      - "Randomize Without Replacement" - will randomly play all before playing a sound twice

### Attenuation Setting 
  - A setting object that can be assigned to Sounds
  - Distance
    - Attenuation Function - Controls how volume changes with distance
  - Spacialization
    - Panning vs Binueral. Panning good for basic sound. Bineural is for rendering for fancy earphones 

## Spawn Sound will spawn an audio component object instance
  - This can be at a Location, Attached to object, or 2D (ie non-spacialized)
  - **Stop** will stop the sound cue
  - **AutoDestroy** the sound component instance to remove it from the game
