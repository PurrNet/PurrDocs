# Connection & Player Events

Most games need to react when someone connects, joins, finishes loading a scene, or leaves. PurrNet raises an event for each of these steps. This page lists them in the order they fire and says where to subscribe.

Most events carry an `asServer` flag. On a host the server and the client each raise their own copy, so a handler subscribed once runs twice: once with `asServer == true` and once with `false`. Check the flag so you only do the work once.

## 1. Connection state

The transport reports its state before PurrNet knows who the player is.

```csharp
networkManager.onServerConnectionState += state => { };
networkManager.onClientConnectionState += state => { };
```

Both are `Action<ConnectionState>`, where the state is `Connecting`, `Connected`, `Disconnecting` or `Disconnected`. The server's state means the server started or stopped. The client's state means this client connected to a server or lost its connection to it. Use these for menus and "connecting..." screens, not for player logic: no `PlayerID` exists yet. `NetworkManager.onAnyServerConnectionState` and `onAnyClientConnectionState` are static versions that fire for every network manager.

## 2. Player joined and left

```csharp
networkManager.onPlayerJoined += (PlayerID player, bool isReconnect, bool asServer) => { };
networkManager.onPlayerLeft += (PlayerID player, bool asServer) => { };
```

`onPlayerJoined` fires only after the connection passed [authentication](authentication.md). A connection that is denied never becomes a player and never fires it. `isReconnect` is `true` when the server has already seen this `PlayerID` during the session, which happens when a client comes back with the same [cookie](README.md#settings-in-the-network-manager).

On the server it fires once for each player that joins. Clients fire it too: when a client joins, it fires for every player already in the game, then for the client itself, then for each player who joins later.

`onPlayerLeft` fires when a player disconnects or is [kicked](README.md#kicking-and-disconnecting-players).

On a client, `networkManager.onLocalPlayerReceivedID` (`PlayerID player`) fires when the server tells this client its own ID. After that, `networkManager.localPlayer` holds it.

These events don't replay. If you subscribe after players have already joined, read `networkManager.players` to get the players that are already in the game.

## 3. Player loaded and unloaded a scene

Joining the server doesn't mean the player has your scene loaded yet. If you want to spawn something for a player, wait for this event:

```csharp
networkManager.onPlayerLoadedScene += (PlayerID player, SceneID scene, bool asServer) => { };
networkManager.onPlayerUnloadedScene += (PlayerID player, SceneID scene, bool asServer) => { };
```

On the server, `onPlayerLoadedScene` fires when a client reports that it finished loading a scene it is part of. On a client it only fires for the local player's own scenes. `onPlayerUnloadedScene` also fires when a player leaves the server while it had the scene loaded.

`onPlayerJoinedScene` and `onPlayerLeftScene` also exist. They track scene membership, which [network rules](network-rules.md) can keep across a reconnect, so they may not fire again. For "the player is ready in this scene", use the loaded/unloaded pair.

## 4. Observer added and removed

Joining and loading are per player. Visibility is per object: a player can see some objects but not others. Override these on a `NetworkBehaviour` to react when a player starts or stops observing that object:

```csharp
protected override void OnObserverAdded(PlayerID player) { }
protected override void OnObserverRemoved(PlayerID player) { }
```

They run on the server only. They are the right place to send a late joiner the state it missed, for example with a `TargetRpc`. See [Network Visibility](network-visibility/README.md) for when they fire.

## 5. Where to subscribe: OnSpawned, not Awake

On a `NetworkBehaviour`, `networkManager` is assigned when the object spawns, so it is still `null` in `Awake`. Subscribe in `OnSpawned` and unsubscribe in `OnDespawned`. The overloads without `asServer` run once even on a host, which matches subscribing once to an event that already carries `asServer`:

```csharp
using PurrNet;

public class PlayerJoinLogger : NetworkBehaviour
{
    protected override void OnSpawned()
    {
        networkManager.onPlayerJoined += OnPlayerJoined;
        networkManager.onPlayerLeft += OnPlayerLeft;

        // These players joined before we subscribed.
        foreach (var player in networkManager.players)
            OnPlayerJoined(player, false, isServer);
    }

    protected override void OnDespawned()
    {
        networkManager.onPlayerJoined -= OnPlayerJoined;
        networkManager.onPlayerLeft -= OnPlayerLeft;
    }

    void OnPlayerJoined(PlayerID player, bool isReconnect, bool asServer)
    {
        if (!asServer)
            return;

        UnityEngine.Debug.Log($"{player} joined (reconnect: {isReconnect})");
    }

    void OnPlayerLeft(PlayerID player, bool asServer)
    {
        if (!asServer)
            return;

        UnityEngine.Debug.Log($"{player} left");
    }
}
```

For a plain `MonoBehaviour` that isn't networked, such as a connection UI, take the manager from `NetworkManager.main` or `InstanceHandler.NetworkManager` and subscribe in `OnEnable`. Unsubscribe in `OnDisable`:

```csharp
using PurrNet;
using PurrNet.Transports;
using UnityEngine;

public class ConnectionStatus : MonoBehaviour
{
    NetworkManager _manager;

    void OnEnable()
    {
        _manager = NetworkManager.main;
        if (_manager)
            _manager.onClientConnectionState += OnClientState;
    }

    void OnDisable()
    {
        if (_manager)
            _manager.onClientConnectionState -= OnClientState;
    }

    void OnClientState(ConnectionState state)
    {
        Debug.Log($"Client connection: {state}");
    }
}
```
