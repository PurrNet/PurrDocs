---
description: Made by network game developers with experience.
---

# 🐈‍⬛ Introduction

### Early release!

PurrNet is still in the early stages! So keep in mind that things could be prone to change, have bugs, or be sub-optimal. We are using it ourselves as well, so we feel confident that it is a ready-to-use system, but it is under constant development and improvement.

Check out the [ROADMAP](introduction/roadmap.md) to see what we are currently planning to add. This can also be a good overview of "what is missing" from the system at this point.

[**Get it on the Unity Asset Store**](https://assetstore.unity.com/packages/slug/297320)\
[**Get it on GitHub**](https://github.com/BlenMiner/PurrNet)

{% embed url="https://youtu.be/VxCPSFBTAms" %}

### Our mission

PurrNet is our attempt at the purrfect networking solution... It's a 100% free Unity Networking solution with no pro or premium version, and no features locked behind a pay-gate. You can use it to release, and we ask nothing in return! Read the [Unique to PurrNet](introduction/unique-to-purrnet.md) section to see what we offer above other solutions!

We have taken inspiration from other networking systems we've worked a lot with in the past like [Mirror](https://mirror-networking.gitbook.io/) and [Fish-Net](https://fish-networking.gitbook.io/). We've tried to keep our version of what the like about these systems, and improve or innovate where we felt needed.

Our mission has been to embrace the natural workflow of Unity rather than fight it, as we've felt other networking solutions do.

Make sure to join our Discord here: [https://discord.gg/NP9tP9Qx9R](https://discord.gg/NP9tP9Qx9R)

Similar to other great networking systems like Mirror and Fish-Net, we also use the same modular transport system, allowing for easy integration to any servers, relays or similar.

Our high-level API, allows for super easy and quick network development, with plenty of freedom for you as the developer. Use our [Network Rule system](systems-and-modules/network-manager/network-rules.md), to define how you prefer your workflow and game to function, from fully [safe/server auth](terminology/server-auth-safe.md), to full [client auth](terminology/client-auth-everyone-unsafe.md) allowing for the quickest multiplayer development!

## Everyone vs Server Auth?

Throughout PurrNet and this knowledge base, you'll find the use of a few words quite often. Most commonly: [Server Auth](terminology/server-auth-safe.md) and "[Everyone](terminology/client-auth-everyone-unsafe.md)".

Server Auth means that only the server can perform the desired action. So if you set the [network rules](systems-and-modules/network-manager/network-rules.md) to use "[Server](terminology/server-auth-safe.md)", only the server can do the action. If you pick "[Everyone](terminology/client-auth-everyone-unsafe.md)" it means that all clients and the server are all allowed to perform the action.

This does technically create an "unsafe" environment, allowing for easy cheating, so this is only recommended for games where cheating is not a major concern. Some good examples are co-op or friendly PvP games.

## Ease of use

The biggest focus of PurrNet is ease of use and scalability for the project. This is greatly involving the "[Network Rules](systems-and-modules/network-manager/network-rules.md)"\
Here are some examples:

#### Spawning & Despawning

With most networking solutions you'd typically need to instantiate an object, follow that with a unique spawn call, and ensure that the instantiation and spawn call occur on the server. However, with PurrNet, you just have to follow your Unity experience and simply Instantiate the object... And that's really it!

With the [network rules](systems-and-modules/network-manager/network-rules.md), you can allow "everyone" to handle spawning, essentially allowing all clients to spawn objects by simply instantiating them. If you only want server auth, you merely instantiate on the server, and the rest is still handled... Easy!

And for despawning? It's the exact same thing! Simply call "Destroy()" as you would in Unity, and the rest is automatically networked for you.

#### [RPC's](systems-and-modules/remote-procedure-call-rpc/) and[ SyncVars](systems-and-modules/network-identity/sync-types/syncvar.md)

With the [Network Rules](systems-and-modules/network-manager/network-rules.md), you can also set up whether the [SyncVar ](systems-and-modules/network-identity/sync-types/syncvar.md)or[ Rpc's ](systems-and-modules/remote-procedure-call-rpc/)allow for [everyone ](terminology/client-auth-everyone-unsafe.md)to call them. This in turn means that you don't have to ensure that the logic runs over the server, like in other networking solutions. This allows for quicker code writing and an easier workflow to learn.

We even have [Generic](systems-and-modules/remote-procedure-call-rpc/generic-rpc.md), [Static ](systems-and-modules/remote-procedure-call-rpc/static-rpc.md)and [Awaitable ](systems-and-modules/remote-procedure-call-rpc/awaitable-rpc.md)RPC's which have not been seen before! (as far as we know)

#### [Ownership](systems-and-modules/network-identity/ownership.md)

When you spawn and despawn things, you can easily modify on a per-object basis who the default [owner ](systems-and-modules/network-identity/ownership.md)of the object is. If the spawning and despawning rules allow [everyone ](terminology/client-auth-everyone-unsafe.md)to spawn, it is easy to also make the spawning player the [owner ](systems-and-modules/network-identity/ownership.md)by simply changing this setting! No extra code is necessary.

Through the [network rules](systems-and-modules/network-manager/network-rules.md), you can also easily allow clients to change the [ownership ](systems-and-modules/network-identity/ownership.md)of an object, allowing for easier use than any other current networking solution!

## Persistent user data and connection

Another great feature of PurrNet is the built-in cookies. This essentially allows for a server's knowledge of a player to remain beyond the connection. If a player disconnects and later connects again, the server is still aware that this is the same player, and can keep the data stored. This is modifiable as to where the data is stored and how persistent it is (Connection, Process, Machine).
