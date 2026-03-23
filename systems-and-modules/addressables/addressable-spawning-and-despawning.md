# Addressable Spawning & Despawning

PurrNet supports spawning networked objects directly from Addressable prefabs. To use this, create an `AddressableNetworkPrefabs` asset via _Create > PurrNet > Network Prefabs > Addressable Network Prefabs_, add your addressable prefab entries to it, and assign it on your NetworkManager.

## Using Addressables.InstantiateAsync

PurrNet automatically intercepts calls to `Addressables.InstantiateAsync` and `AssetReference.InstantiateAsync` via IL weaving. Any instantiated object with a `NetworkIdentity` will be network spawned automatically, just like regular `Instantiate` calls.

This means you can use the standard Addressables API and PurrNet handles the networking for you:

```csharp
public AssetReferenceGameObject prefabReference;

async void SpawnByReference()
{
    var handle = prefabReference.InstantiateAsync(position, rotation);
    await handle.Task;
}
```

You can also use the static `Addressables.InstantiateAsync` with a key or GUID:

```csharp
async void SpawnByGuid()
{
    var handle = Addressables.InstantiateAsync(prefabReference.AssetGUID, position, rotation);
    await handle.Task;
}
```

To despawn and release the instance, use `Addressables.ReleaseInstance` as you normally would. PurrNet intercepts this too and despawns any `NetworkIdentity` on the object before releasing it:

```csharp
Addressables.ReleaseInstance(handle);
```

## Using NetworkManager API

If you prefer explicit control, you can use the `NetworkManager` methods directly. `SpawnAddressableAsync` handles loading the prefab if it hasn't been loaded yet:

```csharp
public AssetReferenceGameObject prefabReference;

async void SpawnObject()
{
    var go = await networkManager.SpawnAddressableAsync(prefabReference);

    if (!go)
        return;

    Debug.Log($"Spawned: {go.name}");
}
```

If you've already pre-loaded your prefabs (via `AddressableNetworkPrefabs.LoadAllAsync`), you can use the synchronous version instead:

```csharp
void SpawnObject()
{
    var go = networkManager.SpawnAddressable(prefabReference);
}
```

Both methods support optional `position`, `rotation`, and `parent` parameters, just like regular instantiation.

To despawn, call:

`networkManager.DespawnAddressable(go);`
