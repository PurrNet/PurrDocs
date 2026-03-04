---
icon: rotate
---

# Lifecycle Events

Network Identity and Network Behaviour come with a set of virtual methods you can override to hook into different points of the network lifecycle. Understanding when these fire is important to building solid multiplayer logic.

## Spawn and Despawn

The most common callbacks are `OnSpawned` and `OnDespawned`. These are called when the object becomes active on the network and when it is removed.

```csharp
public class MyNetworkedScript : NetworkBehaviour
{
    protected override void OnSpawned(bool asServer)
    {
        // Called when this object is first seen by the network.
        // On a host, this fires twice: once for server, once for client.

        if (asServer)
        {
            Debug.Log("Spawned on server side");
        }
        else
        {
            Debug.Log("Spawned on client side");
        }
    }

    protected override void OnDespawned(bool asServer)
    {
        // Called when the object is removed from the network.
        // Same host behavior: fires twice.

        Debug.Log("Despawned");
    }
}
```

There are also parameterless versions if you don't need to know whether you're running as server or client:

```csharp
protected override void OnSpawned()
{
    // Called once regardless of host/server/client
}

protected override void OnDespawned()
{
    // Called once regardless of host/server/client
}
```

### Early Spawn

If you need to run logic before the object receives any synchronized data like [SyncVars](../network-modules/sync-types/syncvar.md) or buffered [RPCs](../remote-procedure-call-rpc/), you can use `OnEarlySpawn`:

```csharp
protected override void OnEarlySpawn(bool asServer)
{
    // Runs before sync data arrives.
    // Good for setting up references or default values.
}
```

## Ownership Changes

You can react to ownership being transferred on a [Network Identity](./):

```csharp
protected override void OnOwnerChanged(PlayerID? oldOwner, PlayerID? newOwner, bool asServer)
{
    Debug.Log($"Owner changed from {oldOwner} to {newOwner}");
}
```

There is also an extended version that tells you whether the change was requested by the local machine:

```csharp
protected override void OnOwnerChanged(PlayerID? oldOwner, PlayerID? newOwner, bool selfRequest, bool asServer)
{
    if (selfRequest)
        Debug.Log("We requested this ownership change");
}
```

### Owner Disconnect and Reconnect

On the server, you can detect when the owner of an object disconnects or comes back:

```csharp
// Server only
protected override void OnOwnerDisconnected(PlayerID ownerId)
{
    Debug.Log($"Owner {ownerId} disconnected");
}

// Server only
protected override void OnOwnerReconnected(PlayerID ownerId)
{
    Debug.Log($"Owner {ownerId} reconnected");
}
```

## Observer Events

Observers are the players who can see a given [Network Identity](./). These callbacks let you react when players start or stop observing:

```csharp
// Server only
protected override void OnObserverAdded(PlayerID player)
{
    Debug.Log($"Player {player} can now see this object");
}

// Server only
protected override void OnObserverRemoved(PlayerID player)
{
    Debug.Log($"Player {player} can no longer see this object");
}
```

There is also a variant that tells you whether the observer is the one who triggered the spawn:

```csharp
protected override void OnObserverAdded(PlayerID player, bool isSpawner)
{
    if (isSpawner)
        Debug.Log("This player caused the object to spawn");
}
```

You can also subscribe to observer changes via events rather than overrides:

```csharp
networkIdentity.onObserverAdded += player => Debug.Log($"{player} added");
networkIdentity.onObserverRemoved += player => Debug.Log($"{player} removed");
```

## Pool Reset

When using [Pooling](pooling.md), this callback fires when the object is returned to the pool. Use it to clean up any state so the object is fresh when reused:

```csharp
protected override void OnPoolReset()
{
    // Reset health, visuals, references, etc.
}
```

## Order of Execution

The general order in which these callbacks fire for a newly spawned object is:

1. `OnEarlySpawn`
2. `OnSpawned`
3. `OnObserverAdded` (for each observing player)
4. `OnOwnerChanged` (if ownership is assigned)

And for despawning:

1. `OnObserverRemoved` (for each observer)
2. `OnDespawned`
3. `OnPoolReset` (if pooling is enabled)
