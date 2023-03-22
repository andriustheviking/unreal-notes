# Unreal Multiplayer

## Table of Contents

- [Resources](#Resources)
- [General](#General) Design
- [Editor](#Editor) Play
- [CLI](#CLI)
- [Dedicated vs Listen Server](#Dedicated-vs-Listen-Server)
- [Object Ownership](#object-ownership)
- [Authority](#Authority)
- [Replication](#Replication)
- [RPCs](#RPCs)
- [Actor Relevancy](#actor-relevancy)
- [Roles](#Roles)
- [Travelling](#Travelling)
- [IOnlineSubsystem](#IOnlineSubsystem)
- [IOnlineSession](#IOnlineSession)
- [Network Failure](#network-failures)

---

# Resources

- [The UE4 Network Compendium](https://assets.teachablecdn.com/pdf_viewer/web/viewer.html?file=https://www.filepicker.io/api/file/b6AoRVWRphvDRp8QZ4jA)
- [Unreal UE4 Networking Overview](https://docs.unrealengine.com/udk/Three/NetworkingOverview.html)

## Platform Online Subsystems

- [Steamworks Notes](./multiplayer-subsystems/steamworks.md)

# General Design Tips

**Never trust the Client!** Server should always test (check) if a client can perform action. *(i.e. does the client have ammo to shoot?)*

#### Debugging tips:

  - Unsuppress `DevNetTraffic` to see logging of all replicated Actors and properties. 
  - The console command `StatNet` is also useful. 
  - It is also useful to use the *Network Profiler* to examine packets that Unreal Engine is sending and receiving.

# Editor Play Modes

- In Editor Play Modes, can select number of players and Net Mode.

- **Net Modes:**
  - Standalone
  - Play as Listen Server
  - Play as Client

# CLI

[Documentation](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/CommandLineArguments/)

- Listen Server: `UEEditor.exe <ProjectName> <MapName>?listen -game`

- Dedicated Server: `UEEditor.exe <ProjectName> <MapName> -server -game -log`

- Client: `UEEditor.exe <ProjectName> <ServerIP> -game`

#### Tips

- To find the server listening port, search for "listening on port" 

### Windows Command Prompt

Server Launch Example:
```
"C:\Program Files\Epic Games\UE_5.0\Engine\Binaries\Win64\UnrealEditor-Cmd.exe" "C:\Users\andri\Workspace\UnrealTutorials\Multiplayer\Multiplayer01\Multiplayer01.uproject"  /Game/ThirdPerson/Maps/ThirdPersonMap?listen?bIsLanMatch=0 -server -log
```

Client Launch Example:
```
"C:\Program Files\Epic Games\UE_5.0\Engine\Binaries\Win64\UnrealEditor-Cmd.exe" "C:\Users\andri\Workspace\UnrealTutorials\Multiplayer\03-SteamSubsystem\PuzzlePlatforms.uproject" -game -log -ResX=1280 -ResY=720 -WINDOWED
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

- `-nosteam` replaces steam subsystem with NULL

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

# Dedicated vs Listen Server

## Dedicated Servers 

- Dedicated servers do not require clients
- Can be run on Virtual Servers that players can connect via fixed IP-Address
- No visual component and no Character representation in-game

## Listen-Server

- A Listen-Server is a Server that is also a Client, so there is always at least one Client connected.

# Object Ownership

Game Architecture is separated between server and its clients into the following general sections:

## Server Only

*These objects only exist on server*

- `AGameMode` 
  - Used to define rules, setup, track points, check win conditions.
  - Will have GameMode for Deathmatch, TeamDeathmatch, etc...
  - Clients will get `nullptr`
  - Manages GameState, replicated to Clients
  - GameMode can consume the **Options** passed via `OpenLevel`, `ServerTravel` or CLI
    - Example: `UGameplayStatics::ParseOptions(OptionsString, "MaxNumPlayers")`
  - **Tip** Check `override` functions to see what it can/should do

## Server & Clients

*These on Server and are replicated to all Clients*

- [`AGameState`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/GameFramework/AGameState/)
  - Use to convey information relevent to all players (ie current score)
  - Useful default properties:
    - `TArray<APlayerState *> `**`PlayerArray`** 
      - Note: The PlayerArray is not directly replicated, but every PlayerState is.
    - `Fname MatchState`
    - `ElapsedTime`

- `APlayerState`
  - The PlayerState is replicated to everyone, accessable via the **GameState.PlayerArray**
  - Useful to store Name, Score, Ping
  - Used to reconnect players and persist them across server travel.
  - Usefule methods to override when Player states are copied
    - `virtual void CopyProperties(class APlayerState* PlayerState)`
    - `virtual void OverrideWith(class APlayerState* PlayerState)`

- `APawn`
  - In this case, the Players' pawn, which is mostly replicated to all clients.
  - Useful methods:
    - `virtual void PossessedBy(AController *NewController)`
    - `virtual void UnPossessed()`

## Server & Owning Client

*Exists on Server and are replicated to a single player's client*

- `APlayerController`

  - RPC is used to pass info from client to server

  - `GetPlayerController(0)`
    - **Listen-Server** - returns the **Listen-Server's** PlayerController
    - **Client** - returns the **Client's** Player Controller
    - **Dedicated Server** - returns the first Client's PlayerController.
    - **NOTE: Other numbers than '0' will not return other Clients for a Client.** This index is meant to be used for local Players (Splitscreen), which we won't cover.

## Owning Client Only

- `AHUD`
  - HUD was used to draw UI before UMG / Widgets. 

- `Widgets` (UMG)

# Authority

Only the server "HasAuthority". This means the Server is the source of truth for the game. 

### bool HasAuthority() 

Returns authority status of UObject. `true` = is server

# Replication

[Unreal Documentation](https://docs.unrealengine.com/5.0/en-US/property-replication-in-unreal-engine/)

Changes to Replicated objects are passed to clients. To replicate, we need `SetReplicates(true)` on the object AND the specified property (ie. `SetReplicateMovement` to have server replicate the object's property to the client.)

**Note:** Only replicated properties are only sent when they change on server. So if the object change on server, but DOES on client, the old properties will NOT update until server side makes some change.

An Actor with `bReplicates` set to true will be spawned and replicated **only when** spawned by the server. A Client spawned actor will be local to that client only.

## Setting Replication

To replicate a property you need to:

1. Make sure you have the `replicated` keyword as one of the parameters to the `UPROPERTY` declaration

```cpp
class ENGINE_API AActor : public UObject
{
    UPROPERTY( replicated )
    AActor * Owner;
};
```

2. In the implementation you need to implement the `GetLifetimeReplicatedProps()` function"

```cpp
void AActor::GetLifetimeReplicatedProps( TArray< FLifetimeProperty > & OutLifetimeProps ) const
{
    DOREPLIFETIME( AActor, Owner );
}
```

### Conditional Replication

`DOREPLIFETIME_CONDITION` declares conditional replication:

```cpp
//Replicates the Variable only to Owner
DOREPLIFETIME_CONDITION(ATestPlayerCharacter, Health, COND_OwnerOnly)
```

Other conditional Keywords:

- `COND_InitialOnly` - This property will only attempt to send on the **initial bunch**
- `COND_OwnerOnly` - This property will only send to the  **Actor's owner**
- `COND_SkipOwner` - This property send to every connection  **EXCEPT**  the owner 
- `COND_SimulatedOnly` - This property will only send to  **simulated**  Actors 
- `COND_AutonomousOnly` - This property will only send to  **autonomous**  Actors 
- `COND_SimulatedOrPhysics` - This property will send to **simulated** OR  **bRepPhysics**  Actors 
- `COND_InitialOrOwner` - This property will send on the  **initial packet** , or to the  **Actor's owner** 
- `COND_Custom` - This property has no particular condition, but wants the  ability to toggle on/off via  **SetCustomIsActiveOverride**

3. In the actor's constructor, make sure you have the bReplicates flag set to true: 

```
  // The new way, inc constructor
  bReplicates = true;
  SetReplicatingMovement(true);
```

*In the above example, The member variable `Owner` will now be synchronized to all connected clients for every copy of this actor type that is currently instantiated (in this case, the base actor class).*

### Tips:

- Avoid having interrelated groups of Actors all replicated, both because of performance and to minimize synchronization issues.
 
 > In Unreal Tournament, Weapons are only replicated to the owning client. Weapon Attachments are not replicated, but spawned client side and controlled through some replicated Pawn properties. Pawns replicate FlashCount and FiringMode, and UT Pawns replicate CurrentWeaponAttachmentClass.

## `RepNotify`

Set a method that gets called when property updated via replication on client.

Header:
```cpp
UPROPERTY(ReplicatedUsing=OnRep_Health)
float Health;

UFUNCTION()
virtual void OnRep_Health(); // Perform logic in implementation that triggers after replication update
```

## Update Frequency

Actors will observe a maximum update frequency set in their `NetUpdateFrequency` variable. This number is the update frequency per second. Setting this based on importance can optimize network performance. 

Example values:

- `10` - Updates every 0.1s for important, unpredictable actors like player controllers in a shooter.
- `5` - Updates every 0.2s for slower moving characters like AI monsters in a coop game.
- `2` - Updates every 0.5s for background actors that are not very important to gameplay, but still need replication.

### Adaptive Network Update Frequency 

With Adaptive Network Update Frequency, we can save CPU cycles that would normally be wasted in redundant attempts to replicate an actor when nothing is really changing. 

**Note:** By default, this feature is deactivated. Setting the console variable `net.UseAdaptiveNetUpdateFrequency` to `1` will activate it

# RPCs

Remote Procedure Calls

[Unreal Documentation](https://docs.unrealengine.com/4.26/en-US/InteractiveExperiences/Networking/Actors/RPCs/)

### Requirements:

1. Must be called from Actors
2. Actor must be replicated
3. RPC from Server to Client will only execute on owning Client Actor
4. RPC from Client to Server can only be called on owning Client Actor
5. Multicast RPCs:
  - Called from Server are executed locally and on all connected Clients
  - Called on Client will only execute locally, and not on Server

## Ownership and RPC

Clients can only call Multicast RPCs on *objects they own.* This is generally limited to the PlayerController their Pawn. *So how do they message to the server?*

We use **Interfaces** to call *input methods* on the PlayerController. Then let the server calculate the target and interaction.  

## Reliable vs Unreliable

### Reliable

**Reliable functions guarantee delivery and order**

They should be used sparingly, as dropped packets will block reliable calls.

### Unreliable

**Unreliable functions make no guarantees about ordering or delivery**

Unreliable functions are sent out if there is room on the connection, and ignored if there is no available bandwidth. Unreliable functions are great for playing sounds, since a late a sound is a wrong sound, it's better to just drop and move on.

### Semi-reliable

Semi-reliable calls for functions that need to maintain order, but can be dropped. 

For example, `ClientAdjustPosition()` will autocorrect if packets are dropped, but out-of-order calls will result in client position jumping around.

## Validation

Validation provides a way to disconnect Client/Server connection from a bad RPC call.

**Implement validation for all RPC functions.** using `WithValidation` keyword:

```cpp
UFUNCTION(Server, unreliable, WithValidation)
void SomeRPCFunction(int32 AddHealth)
```

### Validation Example:

```cpp
bool ATestPlayerCharacter::AddClientHealthRPC_Validate(int32 AddHealth) {
  if (AddHealth > MAX_HEALTH_ADD)  return false; // disconnects the caller

  return true;
}
```

## RPC Implementation

### Server RPC Implementation:

Declare either `reliable` or `unreliable`:

```cpp
// Server RPC marked as reliable and WithValidation
UFUNCTION(Server, reliable, WithValidation)
void Server_GoBoom();
```

Implementing requires `_Implementation` suffix:

```cpp
void ATestPlayerCharacter::Server_GoBoom_Implementation() {
  // BOOM!
}
```

And it also needs a validation definition with `_Validate` suffix:

```cpp
bool ATestPlayerCharacter::Server_GoBoom_Validate() {
  return true;
}
```

### Client RPC Implementation

Client Declaration needs to be marked as `reliable` or `unreliable`:

```cpp
// Client RPC marked as unreliable
UFUNCTION(Client, unreliable)
void MyClientRPCFunction();
```

### Multicast RPC

 also needs to be marked `reliable` or `unreliable`

```cpp
UFUNCTION(NetMulticast, unreliable)
void MyMulticastRPCFunction();
```

# Actor Relevancy and Priority

## Relevancy

Relevancy determines what to replicate to an actor.

`AActor::IsNetRelevantFor()` implements relevancy in the following order:

1. If the Actor's RemoteRole is ROLE_None, then it is not relevant.
2. If the Actor is attached to the skeleton of another actor, then its relevancy is determined by the relevancy of its base.
3. If the Actor has bAlwaysRelevant, then it is relevant.
4. If the Actor has bOnlyRelevantToOwner set to true (used for Inventory), it is only potentially relevant to the client whose player owns that Actor.
5. If the Actor is owned by the player (Owner == Player), then it is relevant.
6. If the Actor is hidden (bHidden = true) and it doesn't collide (bBlockPlayers = false) and it doesn't have an ambient sound (AmbientSound == None) then the actor is not relevant.
7. If the Actor is visible according to a line-of-sight check between the actor's Location and the player's Location, then it is relevant.
8. If the Actor was visible less than 2 to 10 seconds ago (the exact number varies because of some performance optimizations), then it is relevant.

**Note:** Pawn and PlayerController override `AActor::IsNetRelevantFor()`

## Priority

Actors have a float variable `NetPriority`. The higher the number, the more bandwidge it receive. Bandwith is determined by ratio, not absolute.

Priority is calculated with `AActor::GetNetPriority()`

Performance-tuned values are as follows:
- Actor - 1.0
- Pawns - 2.0
- PlayerController - 3.0
- Projectiles - 2.5
- Inventory - 1.4
- Vehicule - 3.0

# Roles

In general, every Actor has a **Role** and a **RemoteRole** property, which describe their authority relationship between the local instance and remote copy

- **Role** describes local authorirt

- **RemoteRole** describes authority of remote copy.

- On the **Server**:

  - All Actors have **Role** set `ROLE_Authority`

  - **RemoteRole** is set to on of the following:
    - `ROLE_AutonomousProxy` for PlayerControllers and Pawns they control
    - `ROLE_SimulatedProxy` for all other replicated Actors
    - `ROLE_None` for Actors not replicated to clients.

- On the **Client** all roles are inversed of the server.

# Travelling

[Documentation](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/Networking/Travelling/)

## Seamless vs Non-Seamless

There are two main ways to travel: **Seamless** and **Non-Seamless**. The main difference, is that seamless travel is a non-blocking operation, while non-seamless will be a blocking call.

When a client executes a non-seamless travel, the client will disconnect from the server and then re-connect to the same server, which will have the new map ready to load.

*It is recommended that UE4 multiplayer games use* **Seamless** *travel when possible.* It will generally result in a smoother experience, and will avoid any issues that can occur during the reconnection process. 

There are three ways in which a non-seamless travel must occur:
1. When loading a map for the first time
2. When connecting to a server for the first time as a client
3. When you want to end a multiplayer game, and start a new one 

### Seamless Transition Map

- Seamless Travel has a **Transition Map** configured in `UGameMapsSettings::TransitionMap` 

- Empty by default

- Transition maps are required due to a Map always needing to be instanced, and Large maps can overload the memory.

- To enable, set `AGameMode::bUseSeamlessTravel = true`

- Can set default transition map in Project Settings

### Seamless Actor Persistence

- Seamless travel can persist some actors. 

- The following are persisted automatically:

  - Server Only:
    - `GameMode` Actor 
      - Any Actors added via `AGameMode::GetSeamlessTravelActorList`
    - All Controllers with valid PlayerState
    - All PlayerControllers
  - Server and Client:
    - All **local** PlayerContrllers
      - Any actors added to `APlayerController::GetSeamlessTravelActorList` added on the **local PlayerController** 

## Travelling Functions

There are three main travelling functions:

- `UEngine::Browse`
- `UWorld::ServerTravel`
- `APlayerController:ClientTravel`

**Note:** When a client loads a Level to use in a networked multiplayer game, it deletes all Actors in the Level except those that have either bNoDelete set to true or bStatic set to true

### UEngine::Browse

- Like a "hard reset"

- Will always result in Non-seamless travel.

- Disconnects Clients from Server

- Dedicated Server cannot travel to other Servers, so the map must be local.

### UWorld::ServerTravel

- [Documentation](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/Engine/UWorld/ServerTravel/)

- Server-only

- `ServerTravel` is performed on the `UWorld` object in the server game instance:
  
  - `bool ServerTravel(const FString& InURL, bool bAbsolute = false, bool bShouldSkipGameNotify = false)` 

- All connected clients will follow

- Example: `World->ServerTravel("/Game/ThirdPerson/Maps/ThirdPersonMap?listen");`
  - loads the map as a listen server

- **NOTE:** The server URL conforms to, and accepts the same arguments as the [CLI](#cli)

### APlayerController::ClientTravel

[Documentation](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/GameFramework/APlayerController/ClientTravel/)

- Can be called from Server or Client:

  - If called from a Client, will travel to a new server

  - If called from Server, will instruct Client to travel to new Map, *but will stay connected to current Server*

- Travel to a different map or IP address. Calls the PreClientTravel event before doing anything. This is implemented as a locally executed wrapper for ClientTravelInternal, to avoid API compatability breakage

- ClientTravel is performed on the player's `PlayerController` instance:
  - `void ClientTravel(const FString& URL, enum ETravelType TravelType, bool bSeamless = false, FGuid MapPackageGuid = FGuid());`

 - Requires `ETravelType`:
```cpp
  enum ETravelType
  {
    TRAVEL_Absolute,   // Absolute URL
    TRAVEL_Partial,    // Partial (carry name, reset server).
    TRAVEL_Relative,   // Relative URL.
    TRAVEL_MAX,
  };
```

# `IOnlineSubsystem`

[Unreal C++ Documentation](https://docs.unrealengine.com/5.1/en-US/API/Plugins/OnlineSubsystem/IOnlineSubsystem/)

[Unreal Documentation](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/Online/)

[Unreal Community Wiki: OnlineSubsystem](https://unrealcommunity.wiki/onlinesubsystem-b4e411)

## Available Services

See [Unreal Wiki: Online Services](https://unrealcommunity.wiki/online-services-f30a0e) for a list of available services.

## Dependency

Requires adding `OnlineSubsystem` to **PublicDependencyModuleNames** in `Source/<ProjectName>/ProjectName.Build.cs`:

```c#
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "GameplayTasks", "OnlineSubsystem" });
```

## Basic Design

The Online Subsystem exists to provide abstraction to common online functionality across available platforms *(Steam, Xbox Live, etc...)*.

When loaded, will try to load the default platform server module specified in `Config/`**`Engine.ini`**:
```
[OnlineSubsystem]
DefaultPlatformService=<Default Platform Identifier>
```
If Successful, the default interface will be available via static accessor:
`static IOnlineSubsystem* Get(FName::Name_None);`

### NULL Subsystem

- Allows you to host LAN, test locally. 

- Engine.ini: `DefaultPlatformService=NULL`

- Returned by `IOnlineSubsystem::Get();`

### Getting a Subsystem

- Subsystems are loading at runtime via `IOnlineSubsystem::Get(FName("Steam"));`

- Custom Subsystem/Master Servers must be built outside of UE

#### Where to Get (Load) the subsystem?

The `UGameInstance::Init` is a valid place to load the subsystem.

### Delegate Design

Every delegate has an `Add`, `Clear` and `Trigger` functions. (**Note:** Manual Triggering is discouraged) 

Common practice is to `Add()` the delegate before calling the appropriate function and then `Clear()` the delegate from within itself

## Interfaces

The following interfaces are included in the Online Subsystem. *Not all platforms implement every Interface, so plan accordingly.*

Primary Interfaces are as follows:

- **Profile:** Anything related to a given User Profile and associated metadata
- **Friends**
- **Sessions:** ([IOnlineSession](#ionlinesession)) Anything related to managing a Session and its state.
- **Shared Cloud:** Interface for sharing files on the cloud
- **User Cloud:** Interface for user cloud file storage
- **Leaderboards**
- **Voice:** Interface for Voice Communication
- **Achievements:** Interface for Reading/Writing/Unlocking Achievments
- **External UI:** Interface to accessing platforms external Interfaces if available.

See [Online Subsystem Documentation](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/Online/) for implementation

# IOnlineSession

- `#include Interfaces/OnlineSessionInterface.h`

- [Overview](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/Online/SessionInterface/)

- [Session Interface](https://docs.unrealengine.com/5.1/en-US/sessions-interface-in-unreal-engine/)

- [Documentation](https://docs.unrealengine.com/4.27/en-US/API/Plugins/OnlineSubsystem/Interfaces/IOnlineSession/)

- Only one SessionInterface will exist at a time: that is the interface for the platform the engine is on. The game interacts with the platform Session via `AGameSession` is the object that wraps the Session Interface, and is how the game calls into it. 

- The GameSession is created and owned by the GameMode, and also only exists on the Server when running an online game. 

- There can be multiple types of GameSessions, but only one instance at a time. This is common for Dedicated Servers

- The Session Settings are defined by the `FOnlineSessionSettings` class.

## `IOnlineSessionPtr IOnlineSubsystem::GetSessionInterface()`

To get the subsystem interface call `GetSessionInterface()` on the subsystem. This will return a `TSharedPtr` `IOnlineSessionPtr`. 

**Note:** Can't forward declare ptr types. If declared in class header, must include `#include "OnlineSubsystem.h"`.

## Sessions and Matchmaking

A session is the instance of the game runnign on a server with a given set of properties. Its either **advertised** or **private**, requiring an invite.

### Session Lifetime

- *Create* a new Session
- *Wait* for Players
- *Register* players
- *Start* the Session
- *Play* the Match
- *End* the session
- *Un-register* players. Ethier:
  - *Update* the session and go back to *Waiting*
  - *Destroy* the session.

## `FOnlineSessionSettings`

- `#include OnlineSessionSettings.h`

- [Documentation](https://docs.unrealengine.com/5.1/en-US/API/Plugins/OnlineSubsystem/FOnlineSessionSettings/)

- Determines characteristics of session. 

- Implements things like 
  - Number of allowed players
  - advertised vs private
  - LAN or not
  - Dedicated or Player-hosted
  - etc...

```cpp
  FOnlineSessionSettings SessionSettings;
  SessionSettings.bIsLANMatch = true;
  SessionSettings.NumPublicConnections = 2;
  SessionSettings.bShouldAdvertise = true;
```

### `Set / Get`

- Use to pass data via SessionSettings search results

- `SessionSettings.Set(FName Key, const ValueType& Value, EOnlineDataAdvertisementType::Type)`
  - Sets a key value pair combination that defines a session setting.
  - `EOnlineDataAdvertisementType::Type`
    - `DontAdvertise`
    - `ViaPingOnly`
    - `ViaOnlineService`
    - `ViaOnlineServiceAndPing` - works over steam service and LAN


## `IOnlineSession::CreateSession`

- Cannot create multiple sessions with the same

- CreateSession requires a delegate to handle completion. 

- Example:
  ```cpp
  // Assign delegate in setup
  SessionInterface->OnCreateSessionCompleteDelegates.AddUObject(this, &UPuzzlePlatformsGameInstance::SessionWasCreated);
  //...
  // Create Session:
  FOnlineSessionSettings SessionSettings;
  SessionInterface->CreateSession(0, TEXT("HostedPuzzleSession"), SessionSettings);
  ```

## `IONlineSession::StartSession`

- Marks the session as in-progress *(as opposed to being in lobby or in progress)*
  - Valid for both Steam and NULL interfaces

## `IOnlineSession::FindSessions`

- [Documentation](https://docs.unrealengine.com/4.26/en-US/API/Plugins/OnlineSubsystem/Interfaces/IOnlineSession/FindSessions/1/)

- `bool FindSessions( int32, const TSharedRef<FOnlineSessionSearch>& )`

- Search results are stored in the `FOnlineSessionSearch` 

- Example:
  ```cpp
  TSharedRef<FOnlineSessionSearch> SessionSearchRef = MakeShared<FOnlineSessionSearch>();
  MySessionInterface->FindSessions(0, SessionSearchRef);
  // Store the ref as a pointer to access on success
  SessionSearchPtr = SessionSearchRef;
  ```

- `void UPuzzlePlatformsGameInstance::OnFindSessionComplete(bool bSuccess)]` 
  - Delegate called when search completes.
  - Results stored in `OnlineSessionSearch` 

### `FOnlineSessionSearch`

- Search results stored in `TArray<FOnlineSessionSearchResult> FOnlineSessionSearch::SearchResults`

- [`FOnlineSessionSearchResult`](https://docs.unrealengine.com/5.1/en-US/API/Plugins/OnlineSubsystem/FOnlineSessionSearchResult/)

- Search result object also contains [`FOnlineSession`](https://docs.unrealengine.com/5.1/en-US/API/Plugins/OnlineSubsystem/FOnlineSession/)

- Example:
```cpp
    FOnlineSessionSearchResult Result = SessionSearch->SearchResults[i];

    FString ServerName = Result.GetSessionIdStr();
    FString HostUserName = Result.Session.OwningUserName;
    uint16 TotalPlayers = Result.Session.SessionSettings.NumPublicConnections;
    uint16 CurrentPlayers = ServerInfo.TotalPlayers - Result.Session.NumOpenPublicConnections;
```

### `FOnlineSession`

- `NumOpenPublicConnections` number of open slots. (Note, NULL subsystem doesn't update this number properly)

- Note: Use the session name enum to name the session: `NAME_GameSession`, or whichever is most appropriate.

## [`IOnlineSession::JoinSession()`](https://docs.unrealengine.com/5.1/en-US/API/Plugins/OnlineSubsystem/Interfaces/IOnlineSession/JoinSession/1/)

- Must join the session via the session interface BEFORE performing Travel()

- Then Travel in the `FOnJoinSessionComplete` delegate call

- Example:
```cpp
  MySessionInterface->JoinSession(0, SESSION_NAME, SessionSearch->SearchResults[searchResultIndex]);
```

### [`FOnJoinSessionComplete`](https://docs.unrealengine.com/5.1/en-US/API/Plugins/OnlineSubsystem/Interfaces/FOnJoinSessionComplete/) delegate

- Pass the ConnectInfo from  `GetResolvedConnectString` to PlayerTravel()

Example:
  ```cpp
  void UPuzzlePlatformsGameInstance::OnJoinSessionComplete(FName SessionName, EOnJoinSessionCompleteResult::Type Result)
  {
    if (Result != EOnJoinSessionCompleteResult::Success) {
      UE_LOG(LogTemp, Error, TEXT("Could not join session. Result=%u"), Result);
      return;
    }

    FString ConnectInfo;
    if (MySessionInterface->GetResolvedConnectString(SessionName, ConnectInfo)) {
      
      APlayerController* PlayerController = GetFirstLocalPlayerController();
      if (!ensure(PlayerController != nullptr)) return;

      UE_LOG(LogTemp, Warning, TEXT("Joining Session %s with connectin info %s"), *SessionName.ToString(), *ConnectInfo);

      PlayerController->ClientTravel(ConnectInfo, ETravelType::TRAVEL_Absolute);
    }
  }
  ```

#### [`IOnlineSession:::GetResolvedConnectString()`](https://docs.unrealengine.com/5.1/en-US/API/Plugins/OnlineSubsystem/Interfaces/IOnlineSession/GetResolvedConnectString/1/)

- `bool GetResolvedConnectString(FName SessionName, FString & ConnectInfo, FName PortType)`

  - `SessionName` the name of the session to resolve
  - `ConnectInfo` the string containing the platform specific connection information
  - `PortType`  type of port to append to result (Game, Beacon, etc)

## Updating Sessions

[`'IOnlineSession::UpdateSession()`](https://docs.unrealengine.com/5.1/en-US/API/Plugins/OnlineSubsystem/Interfaces/IOnlineSession/UpdateSession/) allows you to update the session with `FOnlineSessionSettings`

When the request to update a Session has completed, the `OnUpdateSessionComplete()` delegate is fired.

Updating a Session is generally performed betweenMatches on the Server, but it is also performed on the Client in order to keep the Session information in sync.

## Cloud-Based Matchmaking

Cloud-Based Matchmaking are Matchmaking Services available via a specific platform. For example, Microsoft Xbox Live Service provides matchmaking via the **TruSkill System**

For platforms that support it, call `IOnlinSession::StartMatchmaking()`, and the `'OnMatchmakingComplete` delegate will fire when complete or cancelled.

## Inviting Friends

- `IOnlineSession:FindFriendSession()` to find friend session

  - `OnFindFriendSessionComplete` - delegate

- `IOnlineSession::SendSessionInviteToFriend()` and `IOnlineSession::SendSessionInviteToFriends()` to invite a friend.

  - `OnSessionInviteAccepted` - delegate

# Network Failures

- NetworkFailures are handled by the Game Engine. Basically the session interface sets up the connections, but the engine handles the once set up.

- Can register via the Global `UEngine * GEngine` object:

- Example:
```cpp
// OnNetworkFailure Callback Signature
void UPuzzlePlatformsGameInstance::HandleNetworkFailure(UWorld* World, UNetDriver* NetDriver, ENetworkFailure::Type FailureType, const FString& ErrorString);

// Registering:
if (GEngine != nullptr) // GEngine object can be null
{
  GEngine->OnNetworkFailure().AddUObject(this, &UPuzzlePlatformsGameInstance::HandleNetworkFailure);
}
```