# Spawning & Despawning

When you create or destroy an object in singleplayer, it only exists on your machine. In multiplayer, all connected players need to see the same objects appear and disappear at the same time. Normally, this means you'd need separate "spawn over network" calls, server-side checks, and extra boilerplate every time you want to create an object. PurrNet removes all of that.

Spawning and Despawning in PurrNet is as easy as instantiating and destroying in Unity! If the object contains a network identity (example your own scripts or the prefab link, network transform, etc.), it will automatically be spawned for other clients.

{% embed url="https://youtu.be/XW5T6VKkeCM" %}

You can also easily drag and drop prefabs from your project into the scene in the Unity Editor, and it should just work as well!

Keep in mind that for spawning and despawning the [network rules](../../network-manager/network-rules.md) need to allow your relative role to the object to spawn or despawn it. With the unsafe rules, everyone can do it.

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

## Sending data with the spawn

Sometimes you want an object to arrive already knowing something. A bullet that needs its damage, a pickup that needs its loot table, a character that needs a random seed. You could spawn it and then fire an RPC to fill in the details, but that's a second message and it can land a frame later. Instead you can pack the data straight into the spawn itself.

Override `OnSerialize` to write data on the machine that spawns the object, and `OnDeserialize` to read it back on everyone who receives it. Whatever you write travels with the spawn packet, so on the receiving side it's read back first thing, before `OnEarlySpawn` and `OnSpawned` run. That means you can already rely on the data inside those callbacks.

```csharp
public class Bullet : NetworkIdentity
{
    protected override void OnSerialize(BitPacker packer)
    {
        Packer<string>.Write(packer, "Testing");
    }

    protected override void OnDeserialize(BitPacker packer)
    {
        Debug.Log(Packer<string>.Read(packer));
    }
}
```

Read it back in the exact same order you wrote it, otherwise you'll get garbage. `OnDeserialize` only fires when something was actually written in `OnSerialize`, so an empty override costs you nothing.

This works on [network modules](../../network-modules/) too, so a module can attach its own data to the parent's spawn without the identity needing to know about it.
