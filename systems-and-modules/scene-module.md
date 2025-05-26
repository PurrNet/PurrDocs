---
icon: mountain-sun
---

# Scene Management

Using the Scene Module in PurrNet is as easy as changing scene with Unity's default scene manager. But there is also some more depth to understanding the logic of players in scenes.

Scene management **needs to be handled on the Server**, so everything below is needed to be read with that context!

{% embed url="https://youtu.be/2_15WJCEp7M" %}

## Loading Scenes

For ease of use, you can simply call the `LoadSceneAsync` with the scene name as a string, or the scene id.

```csharp
//Loads scene with default settings
networkManager.sceneModule.LoadSceneAsync("sceneName");
```

There are multiple overloads to the `LoadSceneAsync` method (see image below), such as directly setting the load mode directly, using Unity `LoadSceneParameters`, or adding custom PurrNet settings.

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption><p>Overloads for the LoadSceneAsync</p></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption><p>Overloads for UnloadSceneAsync</p></figcaption></figure>

The [UnloadSceneOptions](https://docs.unity3d.com/ScriptReference/SceneManagement.UnloadSceneOptions.html) are a Unity class, which by default is None when using the UnloadSceneAsync.
