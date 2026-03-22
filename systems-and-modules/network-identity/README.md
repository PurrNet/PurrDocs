---
icon: fingerprint
---

# Network Identity

In multiplayer, the server and all clients need to agree on which objects exist and who they belong to. Every networked object needs some way to be uniquely identified across all machines, so that when the server says "update object X", every client knows exactly which object that is. Without this, there's no way to keep the game state consistent.

The Network Identity is what makes this possible, and it's the core of most networking logic. This plays into the component based nature of object oriented programming with Unity. The [Network Behaviour](networkbehaviour.md) also inherits from Network Identity, meaning that nearly any script interacting on the network, is also a Network Identity.

{% embed url="https://youtu.be/rOWmKKToqEw" %}

You can inherit from Network Identity directly, but it is mostly recommended to have your script inherit from [Network Behaviour](networkbehaviour.md).

Network Identity components can have an owner, and the behavior and control given to said owner will depend on the settings within the [Network Rules](../network-manager/network-rules.md).&#x20;

A single gameobject can have as many Network Identities as you'd want, and [ownership ](ownership.md)doesn't necessarily act on a per gameobject level, but can also be handled on a **per component level**. So one gameobject can have multiple components with different owners. There is a setting on the network identity, to propagate this ownership to the entire object on which it resides. This is also a default setting on the [Network Rules](../network-manager/network-rules.md).

<figure><img src="../../.gitbook/assets/PurrNet execution order(2).jpg" alt=""><figcaption><p>Order of execution within the Network Identity</p></figcaption></figure>
