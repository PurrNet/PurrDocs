---
icon: brain
---

# Network Manager

Every multiplayer game needs a central place that manages the connection, keeps track of who's connected, and holds the settings that define how your networking behaves. Without it, you'd have to wire up connections, transports, prefab registration, and rules all manually across different scripts.

The Network Manager is that central place. It acts as the heart and brain of PurrNet, so all settings of your network are setup through your network manager.

Look at the [Installation/Setup](../../getting-started/installation-setup.md) segment for a guide on the setup.

{% embed url="https://youtu.be/CdkCjyjMF7w?si=VDKglsqtj5PzcQTi" %}

### Settings in the Network manager

**Start Server Flags: These** are the conditions for the server to automatically start upon seeing the NetworkManager

**Start Client Flags:** These are the conditions for the client to automatically start upon seeing the NetworkManager

**Cookie Scope**: PurrNet has cookie/caching for data, and this is where and how the cookies act within PurrNet.\
\- Live With Connection: Nothing is stored once the connection stops\
\- Live With Process: Nothing is stored once the game/process stops\
\- Store Persistently:  Cookies are stored in the player prefs of your system

**Transport:** This is what transport to use for the data sending. By default, you'd likely want to be using the UDP transport. You can just add the transport to the same gameobject and drag it to this field.

**Network Prefabs:** These are the [Network Prefabs](network-prefabs.md) to use. Just simply select the scriptable.

**Network Rules:** These are the [Network Rules](network-rules.md) to use with the Network Manager. Just simply select the scriptable you want to use. This can be overwritten on the individual [network identity](../network-identity/).

**Visibility Rules:** This is the [Visibility ](network-visibility/)rule-set to use. This will decipher the default rules for player to Network Identity relation. This can be overwritten on the individual [network identity](../network-identity/).

## Unreliable messages and the MTU

**MTU Exceeded Behaviour** controls what happens when a message is too large for one packet on an `Unreliable` or `UnreliableSequenced` [channel](../../terminology/channels.md). The setting is under the Network Manager's runtime settings and defaults to **Fragment**.

| Mode | Behavior | Use when |
| --- | --- | --- |
| Fragment | Splits the message into MTU-sized unreliable fragments. The complete message is delivered only if every fragment arrives; missing fragments are not resent. | You want to preserve unreliable delivery and can tolerate losing the entire message. |
| Upgrade To Reliable | Sends the oversized message as `ReliableOrdered` and logs a warning. | Delivery matters more than preserving latency and unreliable semantics. |
| Drop | Discards the oversized message and logs an error. | Oversized messages indicate a mistake and must never silently become reliable or consume fragment bandwidth. |

The MTU is reported by the active transport and can vary by transport, channel, connection, and platform. Avoid designing around one hard-coded byte count.

{% hint style="warning" %}
Fragmentation is a fallback, not a substitute for controlling message size. Each additional fragment is another packet that can be lost. For large data that must arrive intact, use a reliable channel or a purpose-built system such as [SyncBigData](../network-modules/syncbigdata/README.md).
{% endhint %}

You can also inspect or change the mode in code before connecting:

```csharp
using PurrNet.Transports;

networkManager.mtuExceededBehaviour = MTUExceededBehaviour.Fragment;
```

### Per-RPC override

Individual RPCs can override the Network Manager setting through the `mtuExceeded` attribute parameter:

```csharp
[ObserversRpc(channel: Channel.Unreliable, mtuExceeded: MTUBehaviour.Fragment)]
private void SendSnapshot(SnapshotData data) { }
```

`MTUBehaviour.NetworkManager` is the default and follows the global setting. The override is ignored on `UnreliableSequenced`: sequencing is a property of the whole channel, so only the Network Manager setting applies there and the code generator warns about it at build time.

Fragmented messages that fail to complete are counted in the [Bandwidth Profiler](../bandwidth-profiler.md), including the payload type and the reason the message was dropped.
