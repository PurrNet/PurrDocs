# Spawning & Despawning

Spawning and Despawning in PurrNet is as easy as instantiating and destroying in Unity!\
\
If the object contains a network identity (example your own scripts or the prefab link, network transform, etc.), it will automatically be spawned for other clients.

{% embed url="https://youtu.be/XW5T6VKkeCM" %}

You can also easily drag and drop prefabs from your project into the scene in the Unity Editor, and it should just work as well!

Keep in mind that for spawning and despawning the [network rules](network-manager/network-rules.md) need to allow your relative role to the object to spawn or despawn it. With the unsafe rules, everyone can do it.

```csharp
public NetworkIdentity myObject;
private NetworkIdentity _spawnedObject;

private void SpawnMyObject() {
    //This will auto spawn the object
    _spawnedObject = Instantiate(myObject);
    
    //This will spawn it with the ownership
    _spawnedObject.GiveOwnership(localPlayer);
}

private void DespawnMyObject() {
    //Similar to what you do in Unity, just destroy the gameobject and it'll despawn
    if(_spawnedObject)
        Destroy(_spawnedObject.gameObject);
}
```
