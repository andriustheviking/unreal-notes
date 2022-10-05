# Unreal Objects

## Table of Contents

- [GameMode](#gamemode)
  - [GameModeBase](#gamemodebase)
  - [GameMode](#gamemode--gamemodebase)
- [Level](#level)
  - [Level Elements](#level-elements)
- [Light](#light)
  - [Sky light](#sky-light)
  - [Ambient Light](#ambient-light)

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

### Level Elements

- **TriggerBox** - invisible box to trigger events.
  
  - **Tip:** Make visible while editing

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

### Sky Light

- Calculates light from the skybox

- Alternatively can set a default cube map to apply ambient light

### Ambient Light

- Can connect Sky Light and Directional Light so that the Sky light properties accurately reflect the angle of the directional light.
  
  - The skybox will move the sun to where the Direction source is
