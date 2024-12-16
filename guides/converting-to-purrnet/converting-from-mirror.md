# Converting from Mirror

{% hint style="info" %}
Keep in mind that both PurrNet and Mirror are systems which are constantly evolving and could be prone to change.
{% endhint %}

Converting from Mirror to PurrNet fairly simple given that both systems have similar logic for developers.

It is recommended to do the conversion in a separate project in order to use the "old" project for comparison.

{% hint style="warning" %}
Make sure your project is backed up prior to beginning any conversion!
{% endhint %}

## Converting individual parts (& differences)

### Network Manager

Both Mirror and PurrNet rely on a NetworkManager to handle network operations, but their implementations differ. \
This is somewhat what a setup on a gameobject with both systems looks like:\
**Mirror:**

* Network Manager
* Transport
* A list of Spawnable Prefabs
* Built-in callbacks for connection management & registering

**PurrNet:**

* Network Manager
* Transport

To easily setup your Network Manager in just seconds, please have a look at the [Getting Started](../getting-started.md) guide.

### PurrNet Network Identity vs Mirror Network Identity

In both systems, everything that needs to act on the network, must have a Network Identity attached. The difference being that Mirror's network identity acts as a full gameobject networking component. PurrNet's [Network identity](../../systems-and-modules/network-identity/) is what every networked component inherits from, and isn't a component of its own. This is meant to give the ultimate flexibility for the developer(s).

**Comparison**

**Mirror:**

❌ No support for nested networked prefabs.

❌ Single ownership per GameObject.

❌ No runtime un-parenting of transforms.

**PurrNet:**

✔️ Supports nested prefabs.

✔️ Allows split ownership across a single gameobject.

✔️ Enables dynamic transform management at runtime.

### Spawning & Despawning

Handling new objects or removing existing networked objects is a bit different between the systems. In Mirror, you'd need to instantiate the object and spawn the attached NetworkIdentity component. And if you'd want to spawn with ownership, it would be added as a parameter to the spawn call. \
That would look like this:\
**Mirror:**

```csharp
GameObject go = Instantiate(myPrefab);
NetworkServer.Spawn(go, connectionToClient);
NetworkServer.Destroy(go);
```

In PurrNet handles [spawning & despawning](../../systems-and-modules/spawning-and-despawning.md) automatically, however, if you want to modify something relative to the identity, you can do that at the same time as spawning it, and it will arrive in the same packet.\
**PurrNet:**

```csharp
var identity = Instantiate(myIdentityPrefab);
identity.GiveOwnership(ownerPlayer);
Destroy(identity.gameObject);
```

### RPC's

Working with RPC's from **Mirror** to **PurrNet** is pretty easy, as the naming conventions change, but functionality remains. However, there are some key differences in terms of usage. This will depend on your [Network Rules](../../systems-and-modules/network-manager/network-rules.md) so make sure to read up on those. If you are running with a rule set similar to the default "ServerStrict" rules, then the usage will be the same. But if you're running with a rule set similar to the "Unsafe" rules, then you can send any RPC directly from any client!

| Mirror       | PurrNet         |
| ------------ | --------------- |
| \[Command]   | \[ServerRpc]    |
| \[ClientRpc] | \[ObserversRpc] |
| \[TargetRpc] | \[TargetRpc]    |

### SyncTypes

Synchronizing is ab it different between the systems. It's similar in naming and nature. Both systems are meant to automatically sync for you, however, the setup and usage slightly varies.\
\
Other than that, PurrNet also allows for fully owner authorized SyncTypes, making them instantly responsive to it's controller. Read more on this in the individual [SyncType guides](../../systems-and-modules/network-identity/sync-types/).\
\
**Setting up a SyncVar and changing value in Mirror:**

```csharp
[SyncVar]
private string mySyncName;

private void MyMethod(string newName) {
    mySyncName = newName;
}
```

**Setting up a SyncVar and changing value in PurrNet:**

```csharp
private SyncVar<string> mySyncName = new();

private void MyMethod(string newName) {
    mySyncName.value = newName;
}
```
