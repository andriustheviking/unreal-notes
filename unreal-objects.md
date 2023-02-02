# Unreal Functions and Objects

## Table of Contents

- [Unreal Library](#unreal-library)
- [Unreal Math Library](#unreal-math-library)
- [Unreal Objects](#unreal-objects)
  - [TSubclassOf\<T\>](#tsubclassof-t)
  - [Multicast Delegate](#multicast-delegate)
  - [Constructors](#constructors)
  - [Precompiler Macros](#precompiler-macros)
  - [Logging](#logging)
  - [GameFramework Library Objects](#GameFramework-Library-Objects)
  - [Damage](#damage)
- [Kismet Library](#kismet-library)
- [Actors](#actors)
  - [Spawning Actors](#spawn-actors)
  - [Actor](#actor-classes)
  - [Pawn](#apawn--aactor)
  - [Character](#acharacter--apawn)
  - [Controller](#controllers)
- [Camera](#camera)
- [Components](#components)
  - [Component Base Classes](#component-base-classes)
  - [UCharacterMovementComponent](#UCharacterMovementComponent)
-[GameInstance](#gameinstance)
- [GameMode](#gamemode)
  - [GameModeBase](#gamemodebase)
  - [GameMode](#gamemode--gamemodebase)
- [Map](#map)
- [Particles](#particles)
- [Sound](#sound)
- [Timer](#timer)
- [Trace](#trace)
- [Volumes](#volumes)
- [World](#uworld)

---

# Unreal Library

## Helpful Tips
- `UPROPERTY(Meta =(MakeEditWidget = true))` 
  - Adds a widget to the viewport so we don't have to use a USceneComponent.
  - Very handy with FVector properties.
  - **NOTE** the underlying FVector will be object-relative, not world-relative! So (0,0,0) is centered on the object.

## DrawDebugHelpers.h
 - `DrawDebugLine()`
 - `DrawDebugSphere()`

## Asserts

[Documentation](https://docs.unrealengine.com/4.26/en-US/ProgrammingAndScripting/ProgrammingWithCPP/Assertions/)

 - `ensure(Expression)`
  - Notifies the crash reporter on the first time Expression is false. (Very noticible in logs!)
  - Example: ```if (!ensure(TriggerVolume != nullptr)) return;```

## Creating a new Unreal C++ object

To create a new Unreal C++ object that can be accessed in the editor, highlight the **"C++ Classes"** folder in the Content Directory, press **"+ Add"** > **"New C++ Class...**. Unreal will generate the header and implementation files, and hook it up in the project.

## Unreal Types:

## Text types

- Use `Text` variables where localised alpha-numeric display is required
- Use `Name` variables for alpha-numeric network sends/receives, or where a smaller memory footprint is desired
- Use `String` variables for all other alpha-numeric requirements, unless forced to cast, or if you know in advance that you will be using string manipulation functions

- `FName`
  - 8 bytes / character
  - Recommended for sending over network.
  - `Name_None` is FName null equivalent
  - Need to call `.ToString()` to get FString pointer.
  - Dereference FString to get FName

 - `FString`
  - 16 bytes / character
  - `GetName()` returns an object's string name
  - If passing to `%s` token, need to dereference with `*` operator 
  - To create a dynamic FString use `FString::Printf`. Example: `FString::Printf(TEXT("Joining %s"), *Address))`. where Address is type `const FString&`.
  - Example:
  ```cpp
    int32 ServerID = ServerListScrollBox->GetSlots().Num();
    FString ServerNameString = FString::Printf(TEXT("Server %d"), ServerID);
  ```

- `FText`
  - 40 bytes per character
  - Used for visibly displayed text, ie [`UTextBlocks`](./unreal-ui.md)
  - `FText::FromString(SomeString)` to convert FString to FText
  - Example:
  ```cpp
    UTextBlock* ServerName = NewObject<UTextBlock>();   
    ServerName->SetText(FText::FromString(ServerNameString));
  ```

## Unreal Math Library

- [Documentation](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Core/Math/)

### Structs

- `FTransform`
  - [Documentation](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Core/Math/FTransform/)
  - Transform composed of Scale, Rotation (as a quaternion), and Translation. 
  - Transformation of **position** vectors is applied in the order: Scale -> Rotate -> Translate. 
  - Transformation of **direction** vectors is applied in the order: Scale -> Rotate. 
  - Example:
  ```
    StartLocation = GetActorLocation();
    EndLocation = GetTransform().TransformPosition(ObjectRelativeVector);
    MoveVector = (EndLocation - StartLocation) / MoveSpeedSeconds;
  ```

- `FVector` 
  - Vector type_def, can be 2, 3, 4 dimensional
  - FVector (float), also can be int32 typed
  - `GetSafeNormal` - Returns the normalized vector (aka amplitude of 1)

- `FRotator`
  - Similar to 3 dimensional TVector, but with rotation

- `FQuat` - Quaternion (aka Hamiltonion), like a 3D rotator but additional dimension prevents gimbal lock

### Functions

- `Lerp` - Performs Linear Interperlation between to values

# Unreal Objects

### Defining Objects

- Define a new Unreal C++ objects in the Editor, go to **Tools > New C++ Class**. Can be defined from any existing class.

## Unreal Reference Counting

### [TOptional](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Core/Misc/TOptional/)

Struct for *statically allocated optionals*. Since it's static, not a pointer. **Use `IsSet()` in place of *IsValid*.**

- `IsSet()` - returns True if the value is meaningful

- `GetValue()` returns the value

- Assign value with `=` like a normal variable

### [TSharedPtr](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Core/Templates/TSharedPtr/)

- `TSharedPtr` increments and decrements object ref count.

- To check null, must use `.IsValid()` method

- `MakeSharable()` converts a pointer type to sharedPtr. Use `MakeShared` when possible. Otherwise the original pointer and shareptr counter may be fragmented.

### [TSharedRef](https://docs.unrealengine.com/5.1/en-US/shared-references-in-unreal-engine/)

- `MakeShared<T>()` creates a shared ref of type T.

- A Shared Reference acts like a Shared Pointer, but Shared References *must always reference a non-null object*

- Can assign TSharedPtr to TSharedRef if explicitly check ptr `.IsValid()`

### Garbage Collection

- Unreal GC uses the [Unreal Reflection System](https://www.unrealengine.com/en-US/blog/unreal-property-system-reflection), which allows the unreal code to analyze itself.
  - These are included in an object via `#include "FileName.generated.h"` and registered via `UENUM()`, `UCLASS()`, `USTRUCT()`, `UFUNCTION()`, and `UPROPERTY()` macros

- Unreal starts from the **root set**, objects which should be permanent. From there it traverses the **UPROPERTY** pointers of each object. So long as the parent UObject is referenced, UPROPERTY's will be guaranteed alive. Any objects not found during this process are free'd.

- To dynamically add an object that's not referenced via `UPROPERTY()`, use `YourObjectInstance->AddToRoot();`. This would be used when dynamically adding Components to root in C++

- `SetOwner()`, `GetOwner()`  sets/references the object owner in the tree

- `TWeakObjectPtr`/`FWeakObjectPtr` will reference an object but not keep it alive. 
  - Use `IsValid()` to check if object if free'd

## TSubclassOf<T>

- Generic `UClass` type that is a subclass of another Type. 

- Useful to reference classes in C++ which were defined in the Editor. (i.e. Blueprints) 

- Can use this to then call super methods or spawn BP objects

## T * Cast<T>

Unreal implementation of `dynamic_cast` Returns **a pointer of type T**, or `nullptr`. Use `Cast<>` on all Unreal Objects.

## [Delegates](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/)

### Multicast vs Singlecast

- [Multicast delegates](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Multicast/) are stored in many-to-one relationships. 

  - `Broadcast()` executes all delegate callbacks

- Singlecast delegates can only have one assignable callback. 

### Static Delegate

- Static delegates (non-dynamic) do not need to be declared as `UFUNCTIONS()`

### Dynamic Delegate

- Dynamic delegates are exposed via the Blueprint layer, and so they must be declared with **`UFUNCTION()`** macro

- Support [reflection](#Garbage-Collection), can be serialized, their functions can be found by name, and they are slower than regular delegates.

- Declare delecate with the macro, specifying the number of parameters. Example: `DECLARE_DELEGATE_OneParam(DelegateName, Param1Type)`

- Variations of the macros above for multi-cast, dynamic, and wrapped delegates are as follows:
  - `DECLARE_MULTICAST_DELEGATE...`
  - `DECLARE_DYNAMIC_DELEGATE...`
  - `DECLARE_DYNAMIC_MULTICAST_DELEGATE...`
  - `DECLARE_DYNAMIC_DELEGATE...`
  - `DECLARE_DYNAMIC_MULTICAST_DELEGATE...`

- To get the call signature, need to look up the constructor macro in the Unreal header file

- Add Singlecast callback with `BindDynamic()`
  - `ServerRow->OnSelectedWidgetDelegate.BindDynamic(this, &UMainMenu::OnSelectedRow);`

- Add Multicast callback with `AddDynamic(Object, ...)`

- **Example:** For Collision Delegate, first three parameters build the delegate property for UPrimitiveComponent. The rest are the Hit Component delegate signature

```cpp
/**
 * Delegate for notification of blocking collision against a specific component.  
 * NormalImpulse will be filled in for physics-simulating bodies, but will be zero for swept-component blocking collisions.
 */
DECLARE_DYNAMIC_MULTICAST_SPARSE_DELEGATE_FiveParams( FComponentHitSignature, UPrimitiveComponent, OnComponentHit, UPrimitiveComponent*, HitComponent, AActor*, OtherActor, UPrimitiveComponent*, OtherComp, FVector, NormalImpulse, const FHitResult&, Hit );
//... next declaration with differen params
```

## Constructors

- `ConstructionHelpers::FClassFinder<T>`

  - `#include "UObject/ConstructorHelpers.h"`
  - Can use to declare a BP class type by referencing the BP file, where T is the parent class of the BP class.
  - Call from inside constructor 
  - `FClassFinder.Class` Property returns that Class object
  - Example:
  ```cpp
  // In a class's constructor...
  // Initialize a class finder
  ConstructorHelpers::FClassFinder<UUserWidget> MenuWidgetClassFinder(TEXT("/Game/MenuSystem/WBP_MainMenu"));

  if (MenuWidgetClassFinder.Class == nullptr) { /*log and return*/ }

  // Access/Store the class reference with .Class property
  TSubclassOf<UUserWidget> MenuWidgetClass = MenuWidgetClassFinder.Class;

  // Tada! We now have a class type of a BP in C++.
  UE_LOG(LogTemp, Warning, TEXT("Found class %s"), *MenuWidgetClass->GetName());
  ```
**Note:** `/Game/` path points to the `/Project/Content/` dir

- `UObject::CreateDefaultSubobject` 

  - Is only callable in a class constructor. 
  - It takes care of creating an instance of the CDO of the subobject's class, setting its outer class as the caller object, among other things. 
  - The created object then becomes the default object for the property when its object class is instantiated

- `NewObject<T>`

  - Function normally used to instantiate object after engine initialization, like during normal gameplay.
  - Provides several convenience overloads to handle most scenarios.

- `UWorld::SpawnActor<T>`

  - Convenience method to spawn actors in a level with specified location, rotation, spawn collision settings, and checks to ensure it's a spawnable actor class.
  - Is just a convenience wrapper of `NewObject<AActor>`

- `ConstructObject` \[Deprecated\]

  - Use `NewObject<T>` instead

- [stackoverflow reference](https://stackoverflow.com/questions/60021804/unreal-engine-4-different-ways-to-instantiate-the-object)

## Precompiler Macros

### UCLASS()

- Needed for the C++ class to be recognized by the Unreal Engine editor.
- Automatically added if Class is generated by UE Editor

### UFUNCTION()

#### Parameters
- `Exec` - Function can be callable from game console
  - Compatibale with **PlayerControllers** (and their associated **Pawns**), **HUDS**, **CheatManagers**, **GameModes**, **GameInstances**
- `BlueprintCallable` - Exposes functions to BP Editor
- `BlueprintPure` - often pared with `const` keyword on function
- `BlueprintImplementableEvent` - Exposes the object call as an Event in BP Editor
  - Basically a delegate method for the BP Editor to implement, but callable in C++. 
  - Does NOT require implementation in `.cpp` file 
  - **Can't be private**

### UPROPERTY()

Always include for object properties on Unreal classes. Super important for correct object lifecycle in game engine. 

#### Parameters
- `Category="foobar"` - puts the variable in a category name 
- `VisibleAnywhere` - Shows variable value in unreal editor (not editable)
- `VisibleInstanceOnly` - Visible only when instanced (in viewer Details window)
- `VisibleDefaultsOnly` - Visible only in default viewer (in Details window)
- `EditAnywhere` - Exposes variable for editing (has similar InstanceOnly, DefaultsOnly)
- `BlueprintReadWrite` - exposes in blueprint graph (get/set)
- `BlueprintReadOnly` - only exposes get
- `meta=()` - additional properties
  - `AllowPrivateAccess` = “true” use to expose private variable to blueprint
  - `Meta =(MakeEditWidget = true)` adds a widget to the viewport so we don't have to use a USceneComponent. **NOTE:** the underlying FVector will be object-relative, not world-relative! So (0,0,0) is centered on the object.

### Logging

- `UE_LOG(Category, Level, TEXT("Some Text"))`
  
  - [Unofficial Documentation](https://unrealcommunity.wiki/logging-lgpidy6i)
  
    - [Verbosity](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Core/Logging/ELogVerbosity__Type/)
  
  - Level `Fatal` will log, then crash program
  
  - `ulog` - CAPTNCAPS shortcut

- To enable Verbose logging for a categry, open console command in game `~`. Then execute: `Log <LogCategory> Verbose/VeryVerbose`. Where LogCategory is the categroy youwant to enable (ie LogTemp or LogOnline)

## GameFramework Library Objects

## Damage

Two primary event types:

### FPointDamageEvent : FDamageEvent

### FRadiaDamageEvent : FDamageEvent

## `UDamageType`
- `#include "GameFramework/DamageType.h"`
- A DamageType is intended to define and describe a particular form of damage and to provide an avenue for customizing responses to damage from various sources.For example, a game could make a DamageType_Fire set it up to ignite the damaged actor. 
- DamageTypes are never instanced and should be treated as immutable data holders with static code functionality. They should never be stateful.
- Can use `::StaticClass()` to get `TSublclassOf<UDamageType>` to get the damage type received

# EngineUtils

`#include "EngineUtils.h"`

## Functions

- `TActorRange<T>(UWorld *)` - Returns all Actors in a world of a given type

# Kismet Library

## SystemLibrary

`#include "Kismet/KismetSystemLibrary.h"`

- QuitGame()
```cpp
/**Exit the current game 
   * @param SpecificPlayer  The specific player to quit the game. If not specified, player 0 will quit.
   * @param QuitPreference  Form of quitting.
   * @param bIgnorePlatformRestrictions Ignores and best-practices based on platform (e.g on some consoles, games should never quit). Non-shipping only */
  UFUNCTION(BlueprintCallable, Category="Game",meta=(WorldContext="WorldContextObject", CallableWithoutWorldContext))
  static void QuitGame(const UObject* WorldContextObject, class APlayerController* SpecificPlayer, TEnumAsByte<EQuitPreference::Type> QuitPreference, bool bIgnorePlatformRestrictions);
```

## GameplayStatics

`#include "Kismet/GameplayStatics.h"`

- `GetWorldDeltaSeconds(this)`
  - can get DeltaTime outside of Tick function. 
  - **`this`** will pass in the context for the world via the instance’d object

- `GetPlayerPawn(const UObject* WorldContextObject, int32 PlayerIndex)`

- `GetPlayerController(const UObject* WorldContextObject, int32 PlayerIndex)`
  - Note: Player Index only works for local multiplayer

- `ApplyDamage()`
  - will cause *`OnTakeAnyDamage`* to be called called on object
  - Example:
    ```cpp
    UGameplayStatics::ApplyDamage(OtherActor, ProjectileDamage, GetOwner()->GetInstigatorController(), this, UDamageType::StaticClass());
    ```

- `GetAllActorsOfClass(const UObject *WorldContextObject, TSubclassOf<AActor> ActorClass, TArray<AActor*> &OutActors)`

- `SpawnEmitterAtLocation()` / `SpawnEmitterAttached()`
  - See [SpawnEmitter](#SpawnEmitter)

# Actors

### Spawning Actors 
  
#### `SpawnActor<T>(UClass, …)`

  - [Documentation](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/Spawning/ )

  - This creates a new object of class UClass but returns it of type T, *which allows us to create BP class objects and reference them in C++ using C++ declared parent objects.*

  - **Tip:** Use this in conjunction with a property of type `TSubclassOf<T>`, which will then be of type `UClass`. Then assign the class type in UE editor to a BP child of that parent class.
  
  - **Note:** Remember to check against nullptr

  - Example:
  ```cpp
    // BPProjectileClass is a property of type TSubclassOf<AProjectile> which we assign in UE editor.
    AProjectile *BPProjectileInstance = GetWorld()->SpawnActor<AProjectile>(BPProjectileClass, spawnLocation, spawnRotation );
  ```

### Actor Tags

 - Actors contain an array of strings called "Actor Tags"

 - Useful in filtering / selecting actors among sets

 - Example: `SomeActor->ActorHasTag("tagname")`

### DefaultSceneRoot

 - Type `USceneComponent`

 - The component that defines the transform of this Actor in the world, all other components must be attached to this one somehow

 - Assignable as `RootComponent` in C++

## Actor Classes

### AActor
  
  - Base Class. 

  - Store transform information in RootComponent. 

  - Can be replaced with any `USceneComponent` (or derived class)

  - Actors don't have a hierarachy among themselves. Hierarchys are implemented through their components

#### Initialization

 To update on tick, set `PrimaryActorTick.bCanEverTick = true;`

#### Actor Methods

- `virtual void Tick(float DeltaSeconds) override` 

- `AddActorLocalOffset` 
  - Adds a delta to the location of this component in its local reference frame.
  - `bSweep` 
    - Whether to trigger overlaps between current an new locations. 
    - **Only the root component is swept and checked for blocking collision, child components move without sweeping.**
  - `bTeleport`
    - Whether to teleport the physics state (if physics enabled)
    - If true, physics velocity for this object is unchanged (so ragdoll parts are not affected by change in location). If false, physics velocity is updated based on the change in position (affecting ragdoll parts).

- `FTakeAnyDamageSignature OnTakeAnyDamage`
  - **[Multicast Delegate](#multicast-delegate)**
  - Example:
  ```
    // when this object receives damage, the delegate will call our defined method DamageTaken 
    GetOwner()->OnTakeAnyDamage.AddDynamic(this, &UHealthComponent::DamageTaken);
  ```

- `SetOwner()`
  - Actors don't have hierarchy, instead have ownership.
  - Mostly relevent for Damage and Multiplayer

- `GetInstigatorController()` - Needed for `ApplyDamage`

- `SetActorHiddenInGame(bool)` - hides actor

- `SetActorTickEnabled(bool)` - enables/disables tick call

- `SetMobility(EComponentMobility::Type)` - Sets actor movement/mobility. All Actor base classes are Static by default (not moveable).

## APawn : AActor

  Pawn is the base class of all actors that can be possessed by Players or AI. Pawns are controlled by Controllers  

#### Pawn Methods

- `APawn::SetupPlayerInputComponent` 
  - Allows a Pawn to set up custom input bindings. Called upon possession by a [UPlayerController](#player-controller) , using the `InputComponent` created by `CreatePlayerInputComponent()`.
  - implement function override to bind player input to pawn
  - Action Input Events:
  ```cpp
    enum EInputEvent
    {
        IE_Pressed        =0,
        IE_Released       =1,
        IE_Repeat         =2,
        IE_DoubleClick    =3,
        IE_Axis           =4,
        IE_MAX            =5,
    }
  ```
  - `BindAxis()`
    - Example: `PlayerInputComponent->BindAxis(TEXT("MoveForward"), this, &ATank::Move);`
    - NOTE: Joystick axis input will be frame-rate independent. Need to Account for variable framerate

  - `BindAction`
    - Example: `PlayerInputComponent->BindAction(TEXT("Jump"), EInputEvent::IE_Pressed, this, &ACharacter::Jump);`

- `APawn::AddControllerPitchInput(float)` 
  - Look Up
  - Does NOT account for framerate 

- `APawn::AddControllerYawInput(float)` 
  - Look left/right
  - Does NOT account for framerate

- `APawn::AddMovementInput(float)` 
  - Sets the forward direction vector input
  - DOES account for framerate

- `APawn::DetachFromControllerPendingDestroy()`
  - Call this function to detach safely pawn from its controller, knowing that we will be destroyed soon.

## ACharacter : APawn

  Characters are Pawns that have a humanoid mesh, collision, and built-in movement logic. They are responsible for all physical interaction between the player or AI and the world, and also implement basic networking and input models.

  *They are designed for a vertically-oriented player representation that can walk, jump, fly, and swim* using [CharacterMovementComponent](#UCharacterMovementComponent).

- [Documentation](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Engine/GameFramework/ACharacter/)

- The **CharacterMovementComponent** includes useful properties such as Mass, Crouched Half Height, Max Step Heigh, Max Walk Speed, AirControl, etc...

#### Character Movement Functions:

- `ACharacter::Jump(float)`

## Controllers

  Controllers are non-physical Actors that can possess a **Pawn** (or derived class). By default, there is a one-to-one relationship between Controllers and Pawns; meaning, each Controller controls only one Pawn at any given time.

#### Methods:

- `IsPlayerController()`

- `ControlRotation` sets the Pawn rotation

- `GetPlayerViewPoint()`  Returns Player's Point of View. 
  - For the AI this means the Pawn's 'Eyes' ViewPoint
  - For a Human player, this means the Camera's ViewPoint

- `GetPawn()` - returns the pawn controlled by the controller, either player pawn or ai-controlled

- `RestartLevel()`
  - Note: Calling RestartLevel on the PlayerController may encounter a "Travel" error when playing from the editor. Can get around this by playing level as standalone game. 

- AController Interface:
```cpp
  virtual void GameHasEnded(class AActor* EndGameFocus = nullptr, bool bIsWinner = false) override;
  virtual bool IsLocalController() const override;
  virtual void GetPlayerViewPoint(FVector& out_Location, FRotator& out_Rotation) const override;
  virtual void SetInitialLocationAndRotation(const FVector& NewLocation, const FRotator& NewRotation) override;
  virtual void ChangeState(FName NewState) override;
  virtual class AActor* GetViewTarget() const override;
  virtual void BeginInactiveState() override;
  virtual void EndInactiveState() override;
  virtual void FailedToSpawnPawn() override;
  virtual void SetPawn(APawn* InPawn) override;
```

### AI Controllers
  
  See [AI Controllers](./unreal-ai.md#ai-controllers)

### APlayerController
  
  - [Documentation](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/GameFramework/APlayerController/)

  - Accepts player input and issues commands to Player Pawn

  - **Autoposses Player** sets player pawn automatically at beginning of level

  - Generally speaking, UI logic should be implemented on Player Controllers.

  - User the PC to set the input mode (ie mouse for menus)

  **Tip:** It is a good practice to place Character specific Input (Cars work differently than Humans) into your Character/Pawn Classes and to put Input that should work with all Characters or even when the Character Object is not valid, into your PlayerController

#### PlayerController Methods

  - `bShowMouseCursor`

  - `SetInputMode(FInputModeDataBase&)` - sets up input mode

  - `GetFirstLocalPlayerController()` Conveniently get the local PC

  - `GetHitResultUnderCursor()` Gets the hit result under cursor, (i.e. top down ARPG)

  - `SetupPlayerInputComponent()` - binds player input to pawn.
    - Example:
    ```cpp
    // This binds the Move Forward input axis (defined in Project Settings) to call ATank::Move on this
    PlayerInputComponent->BindAxis(TEXT("MoveForward"), this, &ATank::Move);
    ```

  - `virtual GameHasEnded(AActor* EndGameFocus = nullptr, bIsWinner = false) override`

  - `WidgetT* CreateWidget(OwnerT* OwningObject, TSubclassOf<UUserWidget> UserWidgetClass = WidgetT::StaticClass(), FName WidgetName = NAME_None)`
    
    - Can instantiate Blueprint widget in C++ adding a `UPROPERTY TSubclassOf<UUserWidget>` and setting the BP_Widget we created to that property. Then call `CreateWidget(World, GameOverWidgetClass);`

    - UWorld and PlayerControllers are one of the few classes that can act as Widget Owners 

    - Example: `USelectableButton* ServerRow = CreateWidget<USelectableButton>(World, SelectableButtonBPSubclass);`

## TargetPoint

  - An Actor with no default behavior
  - Useful as spawn points

# Camera

- Players' camera must be "activated"

- Best done in Blueprint

### SpringArm

- Controls camera w/r/t player or whatever

- Set Camera Lag and Rotation on springarm to give sense of inertia to camera

- "Use Pawn Control Rotation" automatically sets camera rotation to parent pawn

- Camera should always look downl SpringArm

- "Socket Offset" can offset spring arm origin so an offset camera will connect straight to player (ie over the shoulder). So that in Over the should set up, the spring arm (and camera) compresses toward the player, instead of straight ahead.

### CameraShake

- **Tip:** Easier to create and tweak properties in editer. So create a Blueprint of the camera shake object (ie BP_MyCameraShake)

### MatineeCameraShake : UCameraShakeBase

- Set Oscillation Properties to control shake
  
  - Good starting blueprint values: `Duration=0.25`, `inTime=0.1`, `outTime=0.11`
  - `Location Oscillation` sets amplitude and frequency
  - `Rotation` and `FOV Oscillation` can cause motion sickness

  - **Note:** Camera shaking is best configured in Editor, so we use `UPROPERTY` to define and set the implementation class, which we can then access and call in c++

  - `ClientStartCameraShake()` Example:
    - We reference **MatineeCameraShake** by declaring a `UPROPERTY() TSubclassOf<BaseClass>` and then setting that property in the Editor to the Blueprint object (ie BP_MyCameraShake)
  ```cpp
    // Header
    // We set this UPROPERTY in the editor to a BP class we define and configure for camera shaking
    UPROPERTY(EditAnywhere)
    TSubclassOf<class UCameraShakeBase> MyCameraShakePropertyClass;

    // Implementation
    if (MyCameraShakePropertyClass != nullptr) {
      GetPlayerController()->ClientStartCameraShake(MyCameraShakePropertyClass);
    }
  ```

# Components

## Component Base Classes

  - Components default to NOT update on tick. Must set `PrimaryComponentTick.bCanEverTick = true;` in the class constructor. Set to `false` to save performance, if tick updating not needed.

### UActorComponent : UObject

  - The **Base Component Class** for components that can be added to actors as properties.

  - **NOTE:** *No Tranform* and *No Attachment* capability

  - Useful for simple Components that only hold and manage data (i.e. Health)

### USceneComponent : UActorComponent 

  - Has **Transform** and supports **Attachments**

  - No **Rendering** or **Collision**

#### USceneComponent Methods

  - `SetupAttachment()` 
    - Accepts parent to attach to component. 
    - Should only be called from its Owning Actor's constructor
    - Example:
```
  // In Constructor...
  ProjectileSpawnPoint = CreateDefaultSubobject<USceneComponent>(TEXT("Projectile Spawn Point"));
  ProjectileSpawnPoint->SetupAttachment(RootComponent);
```

  - `AttachToComponent()`
    - Callable at runtime
    - Calling from Actor will use actor's root component

### UPrimitiveComponent : USceneComponent 

  - [Documentation](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Engine/Components/UPrimitiveComponent/)

  - Child of Scene, which also contain or generate geometry.

  - Used to be rendered or used with collision data

#### UPrimitiveComponent Multicast Delegate Methods

  For registering delegates see [Multicast Delegate](#multicast-delegate).

  - `FComponentHitSignature OnComponentHit`
    - **[Multicast Delegate](#multicast-delegate)** *(callback must be a `UFUNCTION()`)*
    - Requires `CollisionEnabledQueryAndPhysics` to be enabled for collision on the calling object
    - For collisions during physics simulation to generate hit events, **Simulation Generates Hit Events** must be enabled for this component. 
    - **Note:** `NormalImpulse` will be filled in for physics-simulating bodies, but will be zero for swept-component blocking collisions.
    - Example binding:
    ```cpp
      BaseMeshComponent->OnComponentHit.AddDynamic(this, &AProjectile::OnHit);
    ```
      - Where `OnHit` is the callback we defined

  - `FComponentBeginOverlapSignature OnComponentBeginOverlap`
    - **Note:** Must have `GetGenerateOverlapEvents()` set to true to generate overlap events.

  - `FComponentEndOverlapSignature OnComponentEndOverlap`  
    - **Note:** Must have `GetGenerateOverlapEvents()` set to true to generate overlap events.

### UCharacterMovementComponent

  - `#include "GameFramework/CharacterMovementComponent.h"`

  - [Documentation](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Engine/GameFramework/UCharacterMovementComponent/)
  
  - To stop an AI controller pawn, `StopMovement()` + `DisableMovement()`. Will stop current movement immediately, and disable acceleration

### UProjectileMovementComponent

  - `#include "GameFramework/ProjectileMovementComponent.h"`

  - [Documentation](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Engine/GameFramework/UProjectileMovementComponent/)

  - Can set on Actor with Tick disabled, which then moves the component *withou* updating the Actor's tick

### UPhysicsHandleComponent

  - `#include "PhysicsEngine/PhysicsHandleComponent.h"`

  - [Documentation](#https://docs.unrealengine.com/5.0/en-US/API/Runtime/Engine/PhysicsEngine/UPhysicsHandleComponent/)

  - ActorComponent Subclass to add interface for grabbability, rotation etc. 

  - **Tip:** Then can get it in C++ using 
      ```
      GetOwner()->FindComponentByClass<UPhysicsHandleComponent >()
      ```

### OtherComponents

  - **[UCapsuleComponent](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Engine/Components/UCapsuleComponent/) : USceneComponent**
    - `#include "Components/CapsuleComponent.h"`

  - **[UStaticMeshComponent](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Engine/Components/UStaticMeshComponent/) : USceneComponent**
    - `#include "Components/StaticMeshComponent.h"`

# Widgets

See [Unreal UI](./unreal-ui.md)

# GameInstance

> High-level manager object for an instance of the running game. Spawned at game creation and not destroyed until game instance is shut down. Running as a standalone game, there will be one of these. Running in PIE (play-in-editor) will generate one of these per PIE instance.

- [Documentation](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/Engine/UGameInstance/)

- `#include "Engine/GameInstance.h"`

- Set the game instance in the project settings

## GameInstance Methods

- `virtual void Init()` 
  - Virtual function to allow custom GameInstances an opportunity to set up what it needs
  - Init for a player when their game starts. 

- `GetEngine()`
  - Returns the engine instance, from which we can call utility functions like `AddOnScreenDebugMessage()`
  - **Note:** This *can* return nullptr, so don't forget to check.

# GameMode

> “GameMode is a blueprint class that controls how a player enters the game. For example, in a multiplayer game, you would use Game Mode to determine where each player spawns. More importantly, the Game Mode determines which Pawn the player will use.”

GameMode typically decides game rules and who won/lost. ie Decides where to spawn Pawns.

Can get GameMode from UWorld via `GetAuthGameMode<T>()`

### GameModeBase

- Base class sufficient for Single Player

- Sets player spawn class for a level

- Can override per-level in World Settings, or for entire game in Project Settings

- **Tip:** Set DefaultGameMode in project settings

- **NOTE:** WorldSettings has a GameMode override, which can interfere with spawn point and "Play From Here" Viewport function.

### GameMode : GameModeBase

- Child class, includes **Match State** and **Multiplayer Matches**

# Map

> Maps (aka Levels) contain other Actors and provide the environment for your game's players.

# Particles

- Can be assigned to a socket on a skeletal mesh
- Can be Activated / Deactivated

## SpawnEmitter

SpawnEmitterAtLocation / SpawnEmitterAttached
  
  - Spawns a **`UParticleSystem`**
  
  - Defaults to spawn one particle effect, but can spawn multiple
  
  - Fire and forget (crreation and deletion is handled)

  - `SpawnEmitterAttached` can attach directly to socket

## UParticleSystem 

  - `#include "Particles/ParticleSystem.h"`

  - [Documentation](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Engine/Particles/UParticleSystem/)

  - **Not a component.**

  - Created with `SpawnEmitterAtLocation`

  - **Note:** Even though an emitter default is spawn once and destroy, if a Particle System is set to repeat, it may never end (i.e. a flame).

## UParticleSystemComponent

  - `#include "Particles/ParticleSystemComponent.h"`

  - [Documentation](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Engine/Particles/UParticleSystemComponent/)

  - Created with `CreateDefaultSubobject`

  - Can attach to a component.

  - Emits particles.

  - Uses a template. 

# UInterface

[Unreal Documentation](https://docs.unrealengine.com/5.1/en-US/interfaces-in-unreal-engine/)
[C++ Interface in Blueprint](https://nerivec.github.io/old-ue4-wiki/pages/interfaces-in-c.html)

**UInterface** operates kind of like Swift/Objective-C `protocols` 

> The UINTERFACE class is not the actual interface. UInterface is an empty class that exists only for visibility to Unreal Engine's reflection system. *The actual interface that will be inherited by other classes must have the same class name, but with the initial "U" changed to an "I".*

**Reference the interface object using the `IMyInterface` class name** The methods are implemented in objects that inheirit from the `IInterface` subclass. This avoids "Diamond Inheirtance"

**NOTE:** Declare the interface methods as *pure virtual* methods by initializing them to 0 in the declaration.

Example:

```cpp
UINTERFACE(MinimalAPI, Blueprintable)
class UReactToTriggerInterface : public UInterface
{
    GENERATED_BODY()
};

class IReactToTriggerInterface
{    
    GENERATED_BODY()

public:
    /** Add interface function declarations here */
  virtual void MyInterfaceMethod() = 0;
};
```

### UINTERFACE

#### MinimalAPI

Causes only the class's type information to be exported for use by other modules. The class can be cast to, but the functions of the class cannot be called (with the exception of inline methods). This improves compile times by not exporting everything for classes that do not need all of their functions accessible in other modules.

#### meta = (CannotImplementInterfaceInBlueprint)

Allows C++ implemented methods to be `BlueprintCallable` without having to use `_Implementation` in their redeclaration!

#### BlueprintType

Exposes the Interface to to Blueprint

#### Blueprintable

If you want Blueprints to be able to implement this interface, you must add the `Blueprintable` metadata specifier to UINTERFACE macro. 

#### NotBlueprintable

Explicitely declares the function is NOT implmented in Blueprint

### UFUNCTION

#### BlueprintCallable

Functions using the `BlueprintCallable` specifier can be called in C++ or Blueprint using a reference to an object that implements the interface.

#### BlueprintImplementableEvent

Functions using `BlueprintImplementableEvent` can not be overridden in C++, but can be overridden in any Blueprint class that implements or inherits your interface.

#### BlueprintNativeEvent

Functions using `BlueprintNativeEvent` can be implemented in C++ by overriding a function with the same name, but with the suffix `_Implementation` added to the end. 

# Sound

  - `#include "Sound/SoundBase.h"`

  - See also [Sounds](./unreal-engine-notes.md#sounds)

  - `#include "Kismet/GameplayStatics.h"`

    - `PlaySoundAtLocation()`

    - `SpawnSoundAtLocation()`

    - `SpawnSoundAttached()`

### USoundBase

  - Sound base class

# Timer

Use `FTimerManager` and `FTimerHandle` to setup timer event scheduling. The FTimer class contains references to world timer functions.

### FTimerHandle

`#include "Engine/EngineTypes.h"`

## TimerManager

`#include "TimerManager.h"`

### FTimerManager

  - `SetTimer(...)` sets the timer

  - Can access via `AActor::GetWorldTimerManager` or `UWorld::GetTimerManager`

### FTimerDelegate

  - Use to bind callbacks with parameters

  - Create with `::CreateUObject()`

  - Example:
  ```cpp
  FTimerDelegate PlayerStartTimerDelegate = FTimerDelegate::CreateUObject(ToonTanksPlayerController, &AToonTanksPlayerController::SetPlayerEnabledState, true);
  // Where `ToonTanksPlayerController` is the object that’s called on
  ```

### FTimer

`#include "RenderCore.h"`

# Trace

- Can be either a **line trace** or **Sweep** (aka shape trace)

- **Note:** By default CharacterClasses ignores trace hits. To fix, change **Collision** of components you want to collide with from **Pawn** to **Custom**. Then change **Visibilty Trace** from **Ignore** to **Block** (or Overlap)

### **LineTraceByChannel**

- Return Bool is the same as OutHit.BlockingHit

#### Trace Channel

- Groups traceability of objects and collision/overlap behaviors

- Can add channels in **Project Settings > Trace Channel**

- Useful channel: `ECollisionChannel::ECC_Visibility`

- Editor / c++ channel name mappings can be found in `[ProjectDirectory]/Config/DefaultEngine.ini`

### LineTraceByObjectType

- Less flexibility than by channel. 

## FCollisionQueryParams

Trace Params to provide more context to the trace

```
FCollisionQueryParams
(
    FName InTraceTag,             // Adds a tag name to the trace
    bool bInTraceComplex = false, // Whether to trace against complex collision 
    const AActor* InIgnoreActor   // Add an Actor to ignore.
)
```

- Methods:
  - `AddIgnoredActor()` - can add multiple actors to ignore via method call.
    - **Note:** Adding an Actor's owner won't immediately ignore the owned actor. Must add them individually.

# Volumes

### TriggerVolumeActor

- Volume to trigger game events

- **Tip:** Make visible while editing

### PostProcessingVolume

- Can apply post processing details to specific volumes in level

# UWorld

- **WorldContextObject** - all instanced objects in a world can be a WorldContextObject, so can pass `this` from any instance to provide the World Context

- **TimeSeconds** - in-game time, changes with pauses, game speed, etc...

- `GetAuthGameMode<T>()` returns the world's gamemode as type T

### UWorld::GetWorld

`UWorld * GetWorld()` - Callable from inside Actor, Components, etc dsnfjkl;ndsajndfjk;lnadsjkl;fnjnk;zxdgfnj
