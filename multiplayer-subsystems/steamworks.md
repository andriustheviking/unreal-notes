# Steamworks

[Partner Website](#https://partner.steamgames.com/)

# Steam

Lobbies need to be in the same download region to show up in Lobby browser. This is set in Settings > Downloads > Download Region

# Packaging Game

To enable steamworks on a package, must include the following:

Go to the packaged game folder `[YourGameName]/Binaries/Win64` (or other platform). In this folder create a file called `steam_appid.txt`. Inside this file just write your app id (`480` if you are using the public one for testing)

# Steam Online Subsystem

[Unreal Documentation](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/Online/Steam/)

[Setup Documentation](https://docs.unrealengine.com/5.1/en-US/online-subsystem-steam-interface-in-unreal-engine/)

The SDK needs to be unzipped and copied to `/YourUnrealEnginePath/Engine/Source/ThirdParty/Steamworks/Steam[Current Version]/sdk`

# Enabling Steam Online Subsystem

## Install Plugin

Add and install the **Online Subsystem Steam** plugin via Unreal 

## Add module dependency

First add the module `OnlineSubsystemSteam` to **PublicDependencyModuleNames** in `Source/<ProjectName>/ProjectName.Build.cs`:

```c#
PublicDependencyModuleNames.AddRange(new string[] { "Core", ..., "OnlineSubsystem", "OnlineSubsystemSteam" });
```

## Configure `/Config/DefaultEngine.ini`

Add the following:
```
[/Script/Engine.GameEngine]
+NetDriverDefinitions=(DefName="GameNetDriver",DriverClassName="OnlineSubsystemSteam.SteamNetDriver",DriverClassNameFallback="OnlineSubsystemUtils.IpNetDriver")

[OnlineSubsystem]
DefaultPlatformService=Steam

[OnlineSubsystemSteam]
bEnabled=true
SteamDevAppId=480

; If using Sessions
; bInitServerOnClient=true

[/Script/OnlineSubsystemSteam.SteamNetDriver]
NetConnectionClassName="OnlineSubsystemSteam.SteamNetConnection"
```

#### NetDriverId

Allows Steam to handle client connections, instead of a VPN for example.

#### SteamDevAppId

`SteamDevAppId=480` is the Dev app ID for "Space Wars". It's the only free App ID. Replace with paid ID.

#### Session vs Lobbies

**Note:** If you are using **Sessions** in your game, add the following under `OnlineSubsystemSteam`:
```
bInitServerOnClient=true 
```
If you are using **Lobbies**, you do not need to set this value.

# Testing

Can't test online subsystem from the editor. Must run executable. 

Can only run one steam online instance at a time.

### PC

Can simply launch executable from Command Prompt or "Launch Game"

### Mac

**Must build package** and add the executable to steam library and launch from within steam.

# Lobby Settings

**Tip:** Open `OnlineSessionInterfaceSteam.cpp` to see desired session setting behaviors.

## Create Session Settings:

```cpp
	SessionSettings.bIsLANMatch = false;
	SessionSettings.NumPublicConnections = 2;
	SessionSettings.bShouldAdvertise = true;
	SessionSettings.bUsesPresence = true; 			// For Steam, this Creates Lobby if True, else Creates Internet Session
	SessionSettings.bUseLobbiesIfAvailable = true;
```

## Search Session Settings

```cpp
	TSharedRef<FOnlineSessionSearch> SessionSearchRef = MakeShared<FOnlineSessionSearch>();
	SessionSearch = SessionSearchRef;
	SessionSearch->bIsLanQuery = false;
	SessionSearch->MaxSearchResults = 100; // Default is 1
	SessionSearch->QuerySettings.Set(SEARCH_PRESENCE, true, EOnlineComparisonOp::Equals);
```