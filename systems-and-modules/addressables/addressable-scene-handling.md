# Addressable Scene handling

PurrNet can load and unload Addressable scenes over the network. When the server loads an addressable scene, all connected clients automatically load the same scene, given the scene is public.&#x20;

Only the server can load and unload scenes.

To load a scene, use the `ScenesModule`:

```csharp
public AssetReference sceneReference;

void LoadScene()
{
    var scenes = networkManager.GetModule<ScenesModule>(true);
    scenes.LoadAddressableSceneAsync(sceneReference, new PurrSceneSettings
    {
        mode = LoadSceneMode.Additive,
        isPublic = true
    });
}
```

You can also load by GUID string if you don't have the AssetReference directly:

```csharp
scenes.LoadAddressableSceneAsync("your-asset-guid-here", settings);
```

To unload, you can either unload by GUID (which unloads all instances of that scene):

```csharp
scenes.UnloadAddressableSceneByGuid(sceneReference.AssetGUID);
```

Or unload a specific instance by its SceneID:

```csharp
if (scenes.TryGetSceneIdByAddressableGuid(sceneReference.AssetGUID, out var sceneId))
    scenes.UnloadSceneAsync(sceneId);
```

You can also listen to events for when addressable scenes start loading or finish:

```csharp
scenes.onAddressableSceneStartLoading += (sceneId, guid, asServer) =>
{
    Debug.Log($"Loading addressable scene: {guid}");
};

scenes.onAddressableSceneLoaded += (sceneId, guid, asServer) =>
{
    Debug.Log($"Addressable scene loaded: {guid}");
};
```
