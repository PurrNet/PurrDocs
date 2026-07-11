---
icon: basketball
---

# Network Rigidbody

This component allows you to very easily utilize the Rigidbody over the network. It helps sync the actual forces of the Rigidbody, and utilizes physics to smoothly align it across the network.

{% hint style="info" %}
If you need physics in multiplayer, you might want to read the [Physics In Multiplayer](../guides/physics-in-multiplayer.md) page
{% endhint %}

{% embed url="https://youtu.be/cCeHzBMch58" %}

### Settings

It's a highly dynamic component, so although the default settings should work well for a lot of cases, it's recommended to familiarize yourself with what the various settings do.

A lot of values behind the scenes are modified dynamically, which is why so many settings are exposed. Things like a flat acceptable error threshold or hard correction threshold, yielded bad results during collision and faster movement, which is why the acceleration settings help to dynamically expand the acceptable ranges during big acceleration changes.

<table><thead><tr><th width="230">Setting</th><th>Effect</th></tr></thead><tbody><tr><td>Owner Auth</td><td>Decides who holds the truth of the simulation. If true, the owning client calculates physics (Client Auth). If false, the server does (Server Auth).</td></tr><tr><td>Sync Velocity Relative To Parent</td><td>When using a real or soft parent, subtracts the parent Rigidbody's linear and angular motion before sending velocity. Enable this for objects moving on rotating or translating platforms.</td></tr><tr><td>Use Parent Frame Position Error</td><td>Measures correction and hard-snap distance in the active real-parent or soft-parent frame. This prevents parent motion by itself from appearing as child position error.</td></tr><tr><td>Interpolation Delay</td><td>How far behind real-time (in seconds) the interpolation target sits. Higher values absorb more network jitter but add visual latency. Default: 0.05s.</td></tr><tr><td>Prediction Factor</td><td>Pushes the target position forward using velocity to compensate for interpolation delay. 0 = no prediction, 1 = compensate for the full interpolation delay, >1 = predict further ahead. The offset is deterministic and identical on all machines.</td></tr><tr><td>Position Strength</td><td>How aggressively the rigidbody chases the target position. Acts as the natural frequency of a critically-damped spring. Higher values = tighter tracking but stiffer feel. The steady-state tracking lag is approximately velocity / strength.</td></tr><tr><td>Correction Range</td><td>The distance over which position correction ramps from zero to full strength. Larger values give softer correction, letting local collisions play out naturally before being pulled back. Smaller values give tighter snapping.</td></tr><tr><td>Rotation Strength</td><td>How aggressively the rigidbody corrects rotation errors. Works the same way as position strength but for angular correction.</td></tr><tr><td>Hard Snap Distance</td><td>If the position error exceeds this distance, the rigidbody teleports to the target instead of using forces. Acts as a safety net for large desync.</td></tr><tr><td>Hard Snap Angle</td><td>If the rotation error (degrees) exceeds this threshold, the rotation snaps instantly instead of using torque. Set to a negative value to disable rotation snapping entirely.</td></tr><tr><td>Acceptable Rotation Error</td><td>Rotation error (degrees) below which rotation correction stops, preventing micro-jitter at rest. Set to a negative value to disable rotation correction entirely.</td></tr><tr><td>Position Change Threshold</td><td>Minimum distance the rigidbody must move before triggering a network update. Prevents unnecessary updates while stationary.</td></tr><tr><td>Rotation Change Threshold</td><td>Minimum angle the rigidbody must rotate before triggering a network update.</td></tr><tr><td>Velocity Stop Threshold</td><td>If both linear and angular velocities are below this value, the object is considered stopped and will cease sending updates.</td></tr></tbody></table>

### Parent-relative correction

For a Rigidbody attached to a moving platform, enable both parent-relative options when you want the child's motion and correction error measured independently of the platform:

```csharp
networkRb.syncVelocityRelativeToParent = true;
networkRb.useParentFramePositionError = true;
```

These options work with a real parent and with `SetSoftParent`. Velocity-relative mode requires a `Rigidbody` on the parent or one of its ancestors; without one, the parent's velocity contribution is treated as zero.

A soft parent uses another Network Identity as the Rigidbody's sync frame without changing the Unity transform hierarchy. Call it on the controlling peer (the owner in owner-auth mode or the server in server-auth mode):

```csharp
networkRb.SetSoftParent(movingPlatformIdentity);

// Return to the frame selected by the Space setting and real Unity parent.
networkRb.ClearSoftParent();
```

The selected soft parent is included in state updates, ownership handoffs, and the initial state sent to late joiners.

### Teleporting

Regular physics operations (`position`, `rotation`, `MovePosition`, `MoveRotation`, `AddForce`, etc.) are synced automatically through the normal state sync and should be used for all regular gameplay movement. For instant repositioning like respawns or portals, use the explicit `TeleportTo` method:

```csharp
// Teleport to a new position and rotation (clears buffer, syncs instantly)
networkRb.TeleportTo(spawnPoint.position, spawnPoint.rotation);

// Teleport to just a new position (keeps current rotation)
networkRb.TeleportTo(respawnPosition);
```

`TeleportTo` differs from setting `position`/`rotation` directly in that it:

* Zeros out linear and angular velocity
* Clears the interpolation buffer on all observers
* Sends a reliable RPC so the teleport is never missed
* Prevents the spring from trying to smoothly chase a position that was meant to be instant

To react when normal correction performs a hard **position** teleport because the error exceeded the configured threshold, subscribe to `onTeleportCorrection`:

```csharp
private void OnEnable()
{
    networkRb.onTeleportCorrection += OnTeleportCorrection;
}

private void OnDisable()
{
    networkRb.onTeleportCorrection -= OnTeleportCorrection;
}

private void OnTeleportCorrection(RigidbodyCorrectionContext context)
{
    Debug.Log($"Corrected from {context.previousPosition} to {context.targetPosition}");
}
```

The callback runs after the hard correction has been applied. Its context includes the previous pose, target pose and velocities, measured errors, and the correction settings used for the decision.

If you're replacing a `Rigidbody` reference with `NetworkRigidbody`, you do **not** need to change any of your existing `MovePosition`, `MoveRotation`, or `AddForce` calls — they work identically. Only use `TeleportTo` when you specifically need an instant, non-physical reposition.

### Position transforms (origin-shifted / large worlds)

By default, an unparented `NetworkRigidbody` puts its raw Unity world-space position on the wire. That is fine for ordinary scenes, but it breaks down in two cases:

* **Origin-shifted worlds** — each peer runs with a different local Unity origin (a common technique for large or streaming worlds), so a raw `Vector3` from one peer is meaningless on another.
* **Very large coordinates** — a 32-bit `float` loses precision long before a large world reaches the limits of double precision.

A **position transform** solves both. It is a pluggable converter, invoked only at the wire boundary, that maps outgoing positions into a shared, peer-agnostic absolute frame (a `double3`) and maps incoming absolute positions back into the receiving peer's Unity world space. The absolute `double3` is what travels on the wire and what the interpolation buffer stores, so a local origin shift needs no extra bookkeeping — the next conversion simply reflects the new offset and every buffered snapshot stays correct.

Only translation is converted; rotation and velocity are already origin-invariant. **Parented** rigidbodies are also unaffected — parent-local coordinates are origin-invariant, so they keep using the legacy path regardless.

Implement `INetworkRigidbodyPositionTransform`:

```csharp
using PurrNet;
using Unity.Mathematics;
using UnityEngine;

public class OriginShiftTransform : MonoBehaviour, INetworkRigidbodyPositionTransform
{
    // Sender side: this peer's Unity world space -> shared absolute frame.
    public double3 ToAbsolute(NetworkRigidbody self, Vector3 localWorldPos)
    {
        var origin = WorldOrigin.Current; // your peer's double3 origin offset
        return new double3(localWorldPos.x, localWorldPos.y, localWorldPos.z) + origin;
    }

    // Receiver side: shared absolute frame -> this peer's Unity world space.
    // Must be the exact inverse of ToAbsolute for the current origin.
    public Vector3 ToLocal(NetworkRigidbody self, double3 absolutePosition)
    {
        var local = absolutePosition - WorldOrigin.Current;
        return new Vector3((float)local.x, (float)local.y, (float)local.z);
    }
}
```

A transform is resolved on spawn, first non-null wins:

1. A runtime override set via `networkRb.SetPositionTransform(transform)`.
2. A sibling component implementing `INetworkRigidbodyPositionTransform` on the same GameObject.
3. The process-wide static fallback `NetworkRigidbody.defaultPositionTransform`.

```csharp
// Apply to every NetworkRigidbody in the process (set this once at startup).
NetworkRigidbody.defaultPositionTransform = new OriginShiftTransform();

// Or override a single instance at runtime; pass null to fall back to the chain.
networkRb.SetPositionTransform(myTransform);
```

{% hint style="warning" %}
The transform must be configured **consistently across all peers** — `ToAbsolute`/`ToLocal` must agree on the absolute frame, and `ToLocal` must be the exact inverse of `ToAbsolute` for the current origin. A sibling component on a shared prefab or the static default both satisfy this; setting it asymmetrically (on some peers only) will make objects snap to the origin.

A runtime override set with `SetPositionTransform` is **not** retained across a despawn/respawn — pooled objects re-run the resolution chain. Prefer a sibling component or the static default if you need it to persist.
{% endhint %}

When no transform is installed, positions travel on the wire exactly as in previous versions (a quantized `CompressedVector3` in the peer's own world space) — the feature is fully opt-in and the legacy path is byte-identical to prior behaviour.
