---
icon: mountain-sun
---

# Scene Management

In a singleplayer game, loading a new scene is straightforward. But in multiplayer, you have a real problem: if one player loads a scene, every other connected player needs to load it too, and they all need to end up in the same state. On top of that, you might want some players in one scene and others in a different one entirely. Without networked scene management, you'd have to manually tell every client which scene to load, track who is where, and handle edge cases like late joiners loading into the wrong place.

PurrNet's Scene Module takes care of all of this for you. Using it is as easy as changing scenes with Unity's default scene manager, but it keeps all your players in sync automatically. There is also some more depth to understanding the logic of players in scenes.

Scene management **needs to be handled on the Server**, so everything below is needed to be read with that context!

{% embed url="https://youtu.be/2_15WJCEp7M" %}

## Loading Scenes

For ease of use, you can simply call the `LoadSceneAsync` with the scene name as a string, or the scene id.

```csharp
//Loads scene with default settings
networkManager.sceneModule.LoadSceneAsync("sceneName");
```

There are multiple overloads to the `LoadSceneAsync` method (see image below), such as directly setting the load mode directly, using Unity `LoadSceneParameters`, or adding custom PurrNet settings.

<figure><img src="../.gitbook/assets/image (11) (1).png" alt=""><figcaption><p>Overloads for the LoadSceneAsync</p></figcaption></figure>

You can also load a scene with custom scene settings called `PurrSceneSettings`, see example below:

```csharp
//Loads scene with custom settings
var settings = new PurrSceneSettings();
settings.isPublic = true; //Default setting - Has all connections automatically switch scene
settings.mode = LoadSceneMode.Single; //Default setting - Unloads all other scenes
networkManager.sceneModule.LoadSceneAsync("sceneName", settings);
```

### PurrSceneSettings

The `PurrSceneSettings` is a struct of setting for your scene loading. This has the following values:

* mode : The Unity [SceneLoadMode](https://docs.unity3d.com/ScriptReference/SceneManagement.LoadSceneMode.html), so whether it should be additive or single
* physicsMode : The Unity [LocalPhysicsMode](https://docs.unity3d.com/ScriptReference/SceneManagement.LocalPhysicsMode.html) to load with
* isPublic : Whether the scene automatically pulls all players into the scene or not

## Unloading scenes

Unloading scenes with PurrNet is as easy as calling the unload of the scene, similar to Unity's own scene management. This will unload the scene for all clients in the scene, as well as the server.

```csharp
networkManager.sceneModule.UnloadSceneAsync("sceneName");
```

<figure><img src="../.gitbook/assets/image (12) (1).png" alt=""><figcaption><p>Overloads for UnloadSceneAsync</p></figcaption></figure>

The [UnloadSceneOptions](https://docs.unity3d.com/ScriptReference/SceneManagement.UnloadSceneOptions.html) are a Unity class, which by default is None when using the UnloadSceneAsync.
