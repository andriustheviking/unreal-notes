# Unreal Objects

## Table of Contents

- [Camera](#camera)
- [GameMode](#gamemode)
  - [GameModeBase](#gamemodebase)
  - [GameMode](#gamemode--gamemodebase)
- [Level](#level)
- [Light](#light)
  - [Sky light](#sky-light)
  - [Ambient Light](#ambient-light)
- [Volume](#volume)

---

# Camera

- Players' camera must be "activated"

## Springarm

- Controls camera w/r/t player or whatever

- Set Camera Lag and Rotation on springarm to give sense of inertia to camera

## CameraShake

- **Tip:** Easier to create and tweak properties in editer. So create a Blueprint of the camera shake object (ie BP_MyCameraShake)

### MatineeCameraShake : UCameraShakeBase

- Set Oscillation Properties to control shake
  
  - Good starting blueprint values: `Duration=0.25`, `inTime=0.1`, `outTime=0.11`
  - `Location Oscillation` sets amplitude and frequency
  - `Rotation` and `FOV Oscillation` can cause motion sickness

- C++
  
  - Can reference **MatineeCameraShake** by declaring a `UPROPERTY()``TSubclassOf<BaseClass>` and then setting that property in the Editor to the Blueprint object (ie BP_MyCameraShake)
    
    #### Example:

```cpp
// Header
UPROPERTY(EditAnywhere)
TSubclassOf<class UCameraShakeBase> MyCameraShakeProperty;
// Implementation (don't forget to check nullptr)
GetPlayerController()->ClientStartCameraShake(MyCameraShakeProperty);
```

# GameMode

> “GameMode is a blueprint class that controls how a player enters the game. For example, in a multiplayer game, you would use Game Mode to determine where each player spawns. More importantly, the Game Mode determines which Pawn the player will use.”

### GameModeBase

- Base class sufficient for Single Player

- Sets player pawn class

- Can override per-level in World Settings, or for entire game in Project Settings

- **Tip:** Set DefaultGameMode in project settings

- **NOTE:** WorldSettings has a GameMode override, which can interfere with spawn point and "Play From Here" Viewport function.

### GameMode : GameModeBase

- Child class, includes **Match State** and **Multiplayer Matches**

# Level

> Levels contain other Actors and provide the environment for your game's players.

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
  
  - Alternatively can add a [PostProcessingVolume](#postprocessingvolume), then go to **Exposure > Metering** and switch from **AutoExposureHistogram** to **AutoExposureBasic**.

### Sky Light

- Calculates light from the skybox

- Alternatively can set a default cube map to apply ambient light

### Ambient Light

- Can connect Sky Light and Directional Light so that the Sky light properties accurately reflect the angle of the directional light.
  
  - The skybox will move the sun to where the Direction source is

# Volume

## TriggerVolumeActor

- Volume to trigger game events
- **Tip:** Make visible while editing

## PostProcessingVolume

- Can apply post processing details to specific volumes in level
