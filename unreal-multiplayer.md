# Unreal Multiplayer

# Resources

[The UE4 Network Compendium](https://assets.teachablecdn.com/pdf_viewer/web/viewer.html?file=https://www.filepicker.io/api/file/b6AoRVWRphvDRp8QZ4jA)
[Unreal UE4 Networking Overview](https://docs.unrealengine.com/udk/Three/NetworkingOverview.html)

# Play Modes

- In Editor Play Modes, can select number of players and Net Mode.

- **Net Modes:**
  - Standalone
  - Play as Listen Server
  - Play as Client

# CLI

[Documentation](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/CommandLineArguments/)

### Tips

- To find the server listening port, search for "listening on port" 

### Windows Command Prompt

Server Launch Example:
```
"C:\Program Files\Epic Games\UE_5.0\Engine\Binaries\Win64\UnrealEditor-Cmd.exe" "C:\Users\andri\Workspace\UnrealTutorials\Multiplayer\Multiplayer01\Multiplayer01.uproject"  /Game/ThirdPerson/Maps/ThirdPersonMap?listen?bIsLanMatch=0 -server -log
```

Client Launch Example:
```
"C:\Program Files\Epic Games\UE_5.0\Engine\Binaries\Win64\UnrealEditor-Cmd.exe" "C:\Users\andri\Workspace\UnrealTutorials\Multiplayer\Multiplayer01\Multiplayer01.uproject" <Server IP/LAN Address>:<Optional Port> -game -log -ResX=1280 -ResY=720 -WINDOWED
```
### URL

- Passing a URL is optional, but must immediately follow the executable name or any mode switch if one is present.

- Can pass URL to map, or an IP Address of server

- To Load specific map, add the following argument: `/Game/<Path from Content to Maps>/<Map Name>`
  - Example: `/Game/ThirdPerson/Maps/ThirdPersonMap`

### CLI Arguments:

- `-log` enables logging

- `-server` Run the game as a server using uncooked content.

- `-game` Launch the game using uncooked content.

- Example:
  - Launch in windowed mode add `-ResX=1280 -ResY=720 -WINDOWED` to the cli

### Options:

The additional options are specified by appending them to the map name or server IP address. **Each option is prefaced by a '?', and can set a value with '='. Starting an option with '-' will remove that option from the cached URL options.**

- `game` Tells the engine what GameInfo class to use (overriding default).

- `name` Tells the engine the Player name to use.

- Server Options:

  - `listen` Specifes the server as a listen server.

  - `bIsLanMatch` Sets whether multiplayer game is on the local network (e.g. bIsLanMatch=0).

  - `bIsFromInvite` Specifies that the player joining was invited.

  - `spectatoronly` Starts the game in spectator mode.

# Authority

Only the server "Has Authority"

### bool HasAuthority() 

Returns authority status of UObject. `true` = is server

# Replication

Changes to Replicated objects are passed to clients. To replicate, we need `SetReplicates(true)` on the object AND the specified property (ie. `SetReplicateMovement` to have server replicate the object's property to the client.)

**Note:** Only replicated properties are only sent when they change on server. So if the object change on server, but DOES on client, the old properties will NOT update until server side makes some change.

### Setting Replication

Before UE 5 this was done in `BeginPlay()`:
```
void AMovingPlatform::BeginPlay()
{
  Super::BeginPlay();            // Always call Super on overriden methods!
  if (HasAuthority()) {          // Can only replicate from server
    SetReplicates(true);        // Tells game this is replicated to client
    SetReplicateMovement(true);  // Tells WHAT to replicate to client
  }
}
```

UE 5+ we can set this in the constructor, and don't have to check Authority
```
  // The new way, inc constructor
  bReplicates = true;
  SetReplicatingMovement(true);
```

# Travelling

[Documentation](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/Networking/Travelling/)

There are two main ways to travel: Seamless and non-seamless. The main difference, is that seamless travel is a non-blocking operation, while non-seamless will be a blocking call.

When a client executes a non-seamless travel, the client will disconnect from the server and then re-connect to the same server, which will have the new map ready to load.

It is recommended that UE4 multiplayer games use seamless travel when possible. It will generally result in a smoother experience, and will avoid any issues that can occur during the reconnection process. 

There are three ways in which a non-seamless travel must occur:
- When loading a map for the first time
- When connecting to a server for the first time as a client
- When you want to end a multiplayer game, and start a new one 

## ServerTravel

[Documentation](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/Engine/UWorld/ServerTravel/)

> Jumps the server to new level.

ServerTravel is performed on the `UWorld` object in the server game instance.

`bool ServerTravel(const FString& InURL, bool bAbsolute = false, bool bShouldSkipGameNotify = false)` 

- UWorld method jumps **the server** to new level.
- Example: `World->ServerTravel("/Game/ThirdPerson/Maps/ThirdPersonMap?listen");`
  - loads the map as a listen server
- Clients will automatically be moved to the new map.
- **NOTE:** The server URL conforms to, and accepts the same arguments as the [CLI](#cli)

## ClientTravel

[Documentation](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/GameFramework/APlayerController/ClientTravel/)

> Travel to a different map or IP address. Calls the PreClientTravel event before doing anything. This is implemented as a locally executed wrapper for ClientTravelInternal, to avoid API compatability breakage

ClientTravel is performed on the player's `PlayerController` instance.

`void ClientTravel(const FString& URL, enum ETravelType TravelType, bool bSeamless = false, FGuid MapPackageGuid = FGuid());`

Need to pass `ETravelType`:
```cpp
enum ETravelType
{
  TRAVEL_Absolute,   // Absolute URL
  TRAVEL_Partial,    // Partial (carry name, reset server).
  TRAVEL_Relative,   // Relative URL.
  TRAVEL_MAX,
};
```