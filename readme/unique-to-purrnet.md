---
description: >-
  A proper integration into the Unity workflow. Work with Unity and not against
  it.
---

# ‼️ Unique to PurrNet

PurrNet has all the core networking features you'd expect: [SyncVars](../systems-and-modules/network-modules/sync-types/syncvar.md), [RPCs](../systems-and-modules/remote-procedure-call-rpc/), [spawning & despawning](../systems-and-modules/network-identity/spawning-and-despawning/), and much more. But beyond the basics, there are a number of things we do differently that we think make a real difference in how you build multiplayer games.

{% embed url="https://youtu.be/JJZY9cI2VqE" %}

### Works with Unity, not against it

A lot of networking solutions force you into a different workflow. Nested prefabs don't work, scene objects need baking, and you have to learn a whole new way of doing things you already know how to do. PurrNet is built to feel like Unity. If you can Instantiate, Destroy, and write MonoBehaviours, you can build multiplayer games with PurrNet.

### No baking

We don't bake components or scene IDs. Scene IDs are calculated at runtime based on hierarchy order, so as long as your scenes match, the IDs match. This means no conflicts in version control, no mysterious bake issues, and no extra build steps.

### No Network Objects

Most networking solutions require a single "Network Object" component per GameObject to identify it across the network. This creates limitations on how you structure your objects. PurrNet doesn't do this. Instead, every [NetworkBehaviour](../systems-and-modules/network-identity/networkbehaviour.md) acts as its own [Network Identity](../systems-and-modules/network-identity/), giving you per-component ownership and much more flexibility in how you build your objects.

### Spawning and despawning

To spawn a networked object, you just call `Instantiate()`. To despawn it, you call `Destroy()`. That's it. PurrNet handles the rest. Even our built-in [object pooling](../systems-and-modules/network-identity/pooling.md) hooks into these same Unity calls, so you don't have to change your workflow at all. If you prefer an explicit spawn call like other systems offer, you can configure that through [Network Rules](../systems-and-modules/network-manager/network-rules.md).

### Network Rules

[Network Rules](../systems-and-modules/network-manager/network-rules.md) let you configure your entire authority model from a single scriptable object. Who can spawn? Who can call RPCs? Who syncs data? You control all of this without touching your game code. This means you can prototype with full client authority and tighten security later, all by swapping a scriptable.

### Network Modules

[NetworkModule](../systems-and-modules/network-modules/) is a base class that lets you create reusable, network-aware components that aren't MonoBehaviours. You can attach them to any NetworkIdentity, which makes it easy to share networking logic across different objects without duplicate code.

### Generic RPCs

You can use generics with [RPCs](../systems-and-modules/remote-procedure-call-rpc/generic-rpc.md). This is great for modular code and tool makers who want to build flexible addons and systems on top of PurrNet.

### Static RPCs

You can call [RPCs on static methods](../systems-and-modules/remote-procedure-call-rpc/static-rpc.md), not just on NetworkBehaviours. This gives you more freedom in how you structure your code.

### Returnable RPCs

[Awaitable RPCs](../systems-and-modules/remote-procedure-call-rpc/awaitable-rpc.md) let you return values from remote calls, similar to how you'd work with async/await in regular C#. Call a method on the server, await the result, and keep going.

### Persistent player data

With PurrNet's [cookie system](../terminology/playerid-client-connection.md), if a player disconnects and reconnects, they keep the same player ID and metadata. The server knows it's the same person without you having to build that yourself.

### Easy testing

At runtime, you can drag and drop any prefab from your [Network Prefabs](../systems-and-modules/network-manager/network-prefabs.md) into the scene and it will automatically be spawned on the network. Deleting a network identity from the hierarchy will despawn it. This makes multiplayer testing in the editor much faster.
