# Unreal Editor

## Table of Contents

- [Miscellaneous](#miscellaneous)
- [AI](#ai)
- [Animation](#animation)
- [Blueprint](#blueprint)
  - [Blueprint Functions](#blueprint-functions)
- [Viewport](#viewport)
  - [Viewport Settings](#viewport-settings)
  - [Brush Editing](#brush-editing)
- [Settings](#settings)
  - [General Settings](#general-settings)
  - [Project Settings](#project-settings)
- [Unreal Motion Graphics](#unreal-motion-graphics)
  - [Widgets](#widgets)
  - [Widget Components](#widget-components)

---

# Miscellaneous:

- In-game input: Clamping axis input can prevent spikes in value that could break collision

- Name types are case insensitive

- FBX (FilmBox Format) is recommended mesh format in UE


# AI

### AI Controller

 - See [AI Controller](./unreal-objects.md#ai-controllers)

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

## Event Graph
  - Can access the AnimationBP inside a Pawn’s EventGraph *via the Pawn’s associated Mesh*
  - **Tip:** In the event graph can get AnimationBlueprint owner pawn and cast to class to access info (ie velocity)

## Animation Montage

You can use Animation Montages (Montages) to combine several Animation Sequences into a single asset and control playback with Blueprints. You can also use Animation Montages to replicate Root Motion animation in network games. 

- [Documentation](https://docs.unrealengine.com/5.0/en-US/animation-montage-in-unreal-engine/)

- Can trigguer animations from inside BP Event Graphs via **Play Montage** node

- Can create by assembling Animations in in content folder

- Animation Montages are assigned **slots** to organize, accessable from inside the **AnimBP**. The slot can be hooked up to an AnimBP AnimGraph between the state machine and the output pose. This will allow the AnimBP to play the montage assocated with the slot

- Can add [**AnimNotifies**](#animnotifies) **Trigger** in the Animation Montage

## AnimaNotifies

- Animation Notifications
- Used to spawn sound effects, particle effects and execute code
- Can add listeners inside an AnimationBlueprint Event Graph

# Blueprint

## Blueprint Functions

- **AddTimeline**
  - Can create keyframes for a specific value over a given timeframe.
  - Has its own timeline editor (looks kind of like After Effects)
  - negative value with **Lerp** will push value past Lerp starting value

- **AddMovementInput**
  - Input speed controlled by [CharacterMovementComponent](./unreal-objects.md#UCharacterMovementComponent)

- **Append** - Use to concatinate strings in BP Editor
- **Interpolate** - Linear Interpolation
- **Select** 
  - Select an enum in BP Editor or for mapping values
  - Useful to map `bool` value to different text ouputs

# Viewport

#### Shortcuts

- **Shift + F1**  - regains mouse while running game

- **F8** - pauses execution while running game

- **F11** - Fullscreen

- **Right Mouse** - Play from here 

- **Ctrl + Alt + Left Click** - drags out box to select elements

- **Alt + S** - Simulate. Plays the viewport, but doesn't posses player pawn

- **Shift + 6** - (Brush Editing) Enable brush editing

- **Shift + B** - (Brush Editing) Selects all sides of a selected shape

- **P** - Toggle Nav Mesh View

### Viewport Settings:

- **Engine Scalability Settings** changes the viewport rendering settings

### Brush Editing:

- Tool to quickly create geometry. 

- **Note:** Changing the Scale will stretch material

- Located in "Place Actors" Window

- **Modes > Brush Editing Mode** will display the brush type window

- Specify brush selection under "Geometry" tab in Details Panel

- "Subtractive" shape located in Brush Settings in Details Panel

- "Create Static Mesh" will turn a selection of brush geo objects into a static mesh object. **Note: The last object selected will be used as the Center Reference***

# Settings

### General Settings

- Settings are written to `.ini` files

- The configuration file hierarchy is read in starting with `Base.ini`, with
  values in later files in the hierarchy overriding earlier values.

- Configuration inheritance:
  
  - `Engine/Config/Base.ini`
    
    - Usually empty
  
  - `Engine/Config/BaseEngine.ini`
  
  - `Engine/Config/[Platform]/base[Platform]Engine.ini`
  
  - `[ProjectDirectory]/Config/DefaultEngine.ini`
    
    - Contains Mappings between editor and source code files
    
    - Use to find mappings between C++ and Editor
    
    - **Example**: The `Grabber` trace channel from the editor can be referenced in C++ as `ECC_GameTraceChannel2`
    
    ```cpp
    +DefaultChannelResponses=(Channel=ECC_GameTraceChannel2,DefaultResponse=ECR_Ignore,bTraceType=True,bStaticObject=False,Name="Grabber")
    ```
  
  - `Engine/Config/[Platform]/[Platform]Engine.ini`
  
  - `[ProjectDirectory]/Config/[Platform]/[Platform]Engine.ini`

- Finally, all project-specific and platform-specific differences are saved out to: `[Project Directory]/Saved/Config/[Platform]/[Category].ini`.

### Project Settings:

- Build:
  
  - When using "Package Project" from the "File" menu in the Unreal editor, the editor will cook, stage and package **ALL CONTENT** in your game (whether it is used by the game or not). This can wind up making your packaged game much bigger than necessary. You can reduce what is cooked by using the `-map=` argument when running BuildCookRun.
  - `bCookAll` (DefaultGame.ini): if true, cook everything in the content directory. This means that if 
    you have an asset that is not referenced by any others, it will still 
    end up in the cook.
  - `bCookMapsOnly` (DefaultGame.ini): **This setting only has any effect if bCookAll is set.** If it is true, "cook all" does not actually cook every asset, just all maps and the assets they reference.

- Maps and Modes:
  
  - Set the default maps here

- Input
  
  - Manages key bindings.
  - **Action** and **Axis** input events

# Unreal Motion Graphics

### Widgets

- **Tip:** `RemoveFromParent` on `self` from the Widget's Event Graph will remove the widget instance from the scene entirely.

- Widget Editer has two parts:
  
  - **Designer** - visual canvas to design widget
  
  - **Event Graph** - Blueprint event graph

### Widget Components

- Widgets are made of Widget Components, like text, boxes, images, etc

- **Is Variable** option in the Designer window exposes a widget component in the event graph

- **Canvas Panel**
  
  - The base canvas for a multi-part widget
  
  - Add it by dragging to the widget hierarchy

- **Anchor Points** 
  
  - Used to maintain component position.
  
  - **Tip:** Set the anchor point for widget components to their nearest corner or screen center

- **Bind** 
  
  - Links component properties to widget variables
