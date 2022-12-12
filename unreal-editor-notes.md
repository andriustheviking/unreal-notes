# Unreal Editor

## Table of Contents

- [Miscellaneous](#miscellaneous)
- [Blueprint](#blueprint)
  - [Blueprint Functions](#blueprint-functions)
- [Viewport](#viewport)
  - [Viewport Settings](#viewport-settings)
  - [Brush Editing](#brush-editing)
  - [Volumes](#volumes)
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

- Can filter viewport to see different attributes of level.

- Useful Viewport filters:
  
  - Lit (Default)
  - Unlit
  - Player Collision
  - Visibility Collission

### Brush Editing:

- Tool to quickly create geometry. 

- **Note:** Changing the Scale will stretch material

- Located in "Place Actors" Window

- **Modes > Brush Editing Mode** will display the brush type window

- Specify brush selection under "Geometry" tab in Details Panel

- "Subtractive" shape located in Brush Settings in Details Panel

- "Create Static Mesh" will turn a selection of brush geo objects into a static mesh object. **Note: The last object selected will be used as the Center Reference***

## Volumes

### Nav Mesh Bounds Volume

See [Nav Mesh Bounds Volume](./unreal-ai.md#nav-mesh)

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

#### Build:

- When using "Package Project" from the "File" menu in the Unreal editor, the editor will cook, stage and package **ALL CONTENT** in your game (whether it is used by the game or not). This can wind up making your packaged game much bigger than necessary. You can reduce what is cooked by using the `-map=` argument when running BuildCookRun.
- `bCookAll` (DefaultGame.ini): if true, cook everything in the content directory. This means that if 
  you have an asset that is not referenced by any others, it will still 
  end up in the cook.
- `bCookMapsOnly` (DefaultGame.ini): **This setting only has any effect if bCookAll is set.** If it is true, "cook all" does not actually cook every asset, just all maps and the assets they reference.

#### Maps and Modes:

- Set the default maps here

#### Input

- Manages key bindings.
- **Action** and **Axis** input events

#### Collision

- Trace Channels
  - When creating a new channel, make sure to go through **Presets** to set Trace type for the new channel as expected. (i.e. Do you want **Invisible Walls** to *block* or *ignore* channel?)
  - Can find corresponding trace channel enum in `DefaultEngine.ini` Config file. Example:
    
    ```
    +DefaultChannelResponses=(Channel=ECC_GameTraceChannel1,DefaultResponse=ECR_Block,bTraceType=True,bStaticObject=False,Name="Bullet")
    ```

# Unreal Motion Graphics

## Widgets

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

## UUserWidget

- Can instantiate widget in C++ (in PlayerController) adding a `UPROPERTY TSubclassOf<UUserWidget>` and setting the BP_Widget we created to that property. Then in the PlayerController.cpp, we create and manage it.  

## Widget Palette

  - Text
  
    - "Size To Content" keeps the bounding box to the size of the text content.