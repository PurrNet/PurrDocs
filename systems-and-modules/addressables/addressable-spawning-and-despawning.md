# Addressable Spawning & Despawning

PurrNet supports spawning networked objects directly from Addressable prefabs. To use this, create an `AddressableNetworkPrefabs` asset via _Create > PurrNet > Network Prefabs > Addressable Network Prefabs_, add your addressable prefab entries to it, and assign it on your NetworkManager.

The simplest way to spawn is with `SpawnAddressableAsync`, which handles loading the prefab if it hasn't been loaded yet:

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

To despawn, simply call:

`networkManager.DespawnAddressable(go);`
