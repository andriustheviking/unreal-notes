# Unreal Editor

## Todo:

* Lookup Rotator vs Rotation

* What is a Controller exactly? How does it work? 

## Miscellaneous:

* In-game input: Clamping axis input can prevent spikes in value that could break collision

* Name types are case insensitive

* FBX (FilmBox Format) is recommended mesh format in UE

## UE Editor Filesystem

* `/Config/DefaultEngine.ini`
  
  * Contains Mappings between editor and source code files
  
  * Use to find mappings between C++ and Editor
  
  * **Example**: The `Grabber` trace channel from the editor can be referenced in C++ as `ECC_GameTraceChannel2`
  
  ```cpp
  +DefaultChannelResponses=(Channel=ECC_GameTraceChannel2,DefaultResponse=ECR_Ignore,bTraceType=True,bStaticObject=False,Name="Grabber")
  ```

## Viewport

* **Shift + F1**  - regains mouse while running game

* **F8** - pauses execution while running game

* **F11** - Fullscreen

* **Right Mouse** - Play from here 

* **Ctrl + Alt + Left Click** - drags out box to select elements

* **Alt + S** - Simulate. Plays the viewport, but doesn't posses player pawn

* **Shift + 6** - (Brush Editing) Enable brush editing

* **Shift + B** - (Brush Editing) Selects all sides of a selected shape

<br>Viewport Settings:

- **Engine Scalability Settings** changes the viewport rendering settings

<br>Brush Editing:

* Tool to quickly create geometry. 

* **Note:** Changing the Scale will stretch material

* Located in "Place Actors" Window

* **Modes > Brush Editing Mode** will display the brush type window

* Specify brush selection under "Geometry" tab in Details Panel

* "Subtractive" shape located in Brush Settings in Details Panel

* "Create Static Mesh" will turn a selection of brush geo objects into a static mesh object. **Note: The last object selected will be used as the Center Reference***

## Project Settings
