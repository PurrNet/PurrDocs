# SyncLazyRef

A SyncLazyRef is a synchronized reference to a NetworkIdentity that resolves lazily. Unlike a regular `SyncVar<NetworkIdentity>`, the target does not need to be registered on the receiving client at the moment the reference arrives.

If the target is not yet available (for example, its scene has not loaded, or it has not been spawned yet), the reference will resolve automatically once the target shows up. This is particularly useful for cross-scene references.

Below is a basic usage example:

```csharp
// Reference to any NetworkIdentity
private SyncLazyRef<NetworkIdentity> _target = new();

// Reference to a specific NetworkIdentity subtype
private SyncLazyRef<Player> _player = new();

protected override void OnSpawned(bool asServer)
{
    _target.onChanged += OnTargetChanged;

    if (asServer)
        _target.value = someIdentity;
}

private void OnTargetChanged(NetworkIdentity oldValue, NetworkIdentity newValue)
{
    if (newValue)
        Debug.Log($"Target resolved: {newValue.name}");
    else
        Debug.Log("Target cleared or not yet resolved");
}
```

### How it resolves

Under the hood, SyncLazyRef stores a `GlobalNetworkID`, which pairs the target's scene and NetworkID. When that value arrives on a remote client:

* If the target already lives in a loaded scene on that client, it resolves immediately.
* If not, the reference waits. Once the target spawns, the reference resolves and `onChanged` fires.
* If the target is despawned or its scene unloads, the reference clears back to null. If it shows up again later, it re-resolves without any extra work.

### Setting to null

Assigning null clears the reference on all clients:

```csharp
_target.value = null;
```

### Owner auth

Owner auth is supported the same way as SyncVar:

```csharp
// True for owner auth, false (default) for server auth
private SyncLazyRef<Player> _player = new(ownerAuth: true);
```

### Notes

The target must be spawned at the moment you assign it. Assigning an unspawned identity logs an error and the assignment is ignored.

If the resolved identity is not of type `T`, the reference clears to null and an error is logged.
