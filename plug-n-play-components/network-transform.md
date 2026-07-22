---
icon: maximize
---

# Network Transform

Network Transform synchronizes a GameObject's position, rotation, scale, and optional parent. Add it to the object that should move over the network, then choose which peer has authority and which transform properties should be synchronized.

## Settings

| Setting | Effect |
| --- | --- |
| Sync Position | `No`, `Local`, or `World`. Local positions are relative to the active parent; world positions use world space. |
| Sync Rotation | `No`, `Local`, or `World`, with the same space rules as position. |
| Sync Scale | Synchronizes `localScale`. |
| Sync Parent | Synchronizes `SetParent` changes when the new parent has a Network Identity. |
| Force Sleep Rigidbody | Forces an attached non-controller Rigidbody to sleep so physics does not fight received transform updates. |
| Interpolate Settings | Chooses whether received position, rotation, and scale values are smoothed. |
| Min Buffer Size | Minimum number of ticks retained for interpolation. |
| Max Buffer Size | Maximum number of ticks retained for interpolation. A larger range absorbs more jitter but can add visual delay. |
| Character Controller Patch | Temporarily coordinates a 3D Character Controller while applying received transforms. It can cause physics callbacks to run more than once. |
| Owner Auth | When enabled, the owning client controls the transform. When disabled, the server controls it. |
| Interpolation Timing | Applies interpolation during the selected Unity update phase. Match this to the system that reads or moves the transform. |
| Adaptive Sync | Reduces the send rate while motion can be reconstructed accurately. See [Adaptive sync](network-transform/adaptive-sync.md). |

<figure><img src="../.gitbook/assets/image (4) (1) (1).png" alt="Network Transform inspector"><figcaption><p>The Network Transform component</p></figcaption></figure>

## Delivery and packet loss

Regular transform updates are batched and sent over an unreliable channel to avoid reliable-channel backlog during continuous movement. Current state is delta-compressed against receiver-acknowledged baselines. If a packet is lost, a later update can recover from an acknowledged state instead of permanently applying a delta to the wrong baseline.

PurrNet sends authoritative full-state updates when needed, such as when an observer is added, control changes, or you call `ForceSync`. In normal use, you do not need to manually resend transforms after packet loss.

`ForceSync` sends the latest complete state and bypasses the normal compression optimizations:

```csharp
// Send a full state to every observer.
networkTransform.ForceSync();

// Send a full state to one player.
networkTransform.ForceSync(targetPlayer);
```

Use it for exceptional state changes or recovery logic, not every frame.

{% hint style="info" %}
Network Transform keeps its regular updates below the transport MTU. The Network Manager's unreliable fragmentation setting mainly matters to other oversized PurrNet messages; transform batching is already MTU-aware.
{% endhint %}
