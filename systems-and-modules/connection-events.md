---
icon: plug
---

# Connection Events

The Network Manager exposes a number of events that let you react to connections, disconnections and player activity. These are useful for things like updating player lists, showing join/leave messages, or cleaning up when someone drops.

## Server and Client State

You can listen for when the server or client starts and stops by subscribing to the connection state events:

```csharp
private void Awake()
{
    var nm = InstanceHandler.NetworkManager;
    nm.onServerConnectionState += OnServerState;
    nm.onClientConnectionState += OnClientState;
}

private void OnDestroy()
{
    var nm = InstanceHandler.NetworkManager;
    nm.onServerConnectionState -= OnServerState;
    nm.onClientConnectionState -= OnClientState;
}

private void OnServerState(ConnectionState state)
{
    Debug.Log($"Server is now: {state}");
}

private void OnClientState(ConnectionState state)
{
    Debug.Log($"Client is now: {state}");
}
```

There are also static variants if you need to listen from anywhere without a direct reference to the manager:

```csharp
NetworkManager.onAnyServerConnectionState += OnServerState;
NetworkManager.onAnyClientConnectionState += OnClientState;
```

## Player Joined and Left

These are probably the events you'll reach for most. They fire whenever a player connects or disconnects from the server:

```csharp
private void Awake()
{
    var nm = InstanceHandler.NetworkManager;
    nm.onPlayerJoined += OnPlayerJoined;
    nm.onPlayerLeft += OnPlayerLeft;
}

private void OnDestroy()
{
    var nm = InstanceHandler.NetworkManager;
    nm.onPlayerJoined -= OnPlayerJoined;
    nm.onPlayerLeft -= OnPlayerLeft;
}

private void OnPlayerJoined(PlayerID player, bool isReconnect)
{
    Debug.Log($"Player {player} joined! Reconnect: {isReconnect}");
}

private void OnPlayerLeft(PlayerID player, bool isReconnect)
{
    Debug.Log($"Player {player} left! Will reconnect: {isReconnect}");
}
```

## Local Player ID

If you need to know when the local client has been assigned its [PlayerID](../terminology/playerid-client-connection.md), you can listen for that as well:

```csharp
InstanceHandler.NetworkManager.onLocalPlayerReceivedID += OnLocalPlayerReady;

private void OnLocalPlayerReady(PlayerID localPlayer)
{
    Debug.Log($"We are player: {localPlayer}");
}
```

## Scene Events

When using the [Scene Module](scene-module.md), you can also track when players move between scenes:

```csharp
var nm = InstanceHandler.NetworkManager;

nm.onPlayerJoinedScene += (player, scene) =>
{
    Debug.Log($"Player {player} entered scene {scene}");
};

nm.onPlayerLeftScene += (player, scene) =>
{
    Debug.Log($"Player {player} left scene {scene}");
};

nm.onPlayerLoadedScene += (player, scene) =>
{
    Debug.Log($"Player {player} finished loading scene {scene}");
};
```

## Checking State Directly

Beyond events, you can always check the current connection state through properties on the Network Manager:

```csharp
var nm = InstanceHandler.NetworkManager;

if (nm.isServer) { /* running as server */ }
if (nm.isClient) { /* running as client */ }
if (nm.isHost)   { /* running as both */ }

// Get all currently connected players
var players = nm.players;
var count = nm.playerCount;

// Get our own player ID
var me = nm.localPlayer;
```

## Useful Properties

| Property | Type | Description |
| --- | --- | --- |
| `isServer` | `bool` | Whether this instance is running as a server |
| `isClient` | `bool` | Whether this instance is running as a client |
| `isHost` | `bool` | Whether this instance is both server and client |
| `isOffline` | `bool` | Whether neither server nor client is running |
| `localPlayer` | `PlayerID` | The local player's network ID |
| `playerCount` | `int` | Number of connected players |
| `players` | `IReadOnlyList` | List of all connected players |
