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

<table><thead><tr><th width="230">Setting</th><th>Effect</th></tr></thead><tbody><tr><td>Owner Auth</td><td>Decides who holds the truth of the simulation. If true, the owning client calculates physics (Client Auth). If false, the server does (Server Auth).</td></tr><tr><td>Interpolation Delay</td><td>How far behind real-time (in seconds) the interpolation target sits. Higher values absorb more network jitter but add visual latency. Default: 0.05s.</td></tr><tr><td>Prediction Factor</td><td>Pushes the target position forward using velocity to compensate for interpolation delay. 0 = no prediction, 1 = compensate for the full interpolation delay, >1 = predict further ahead. The offset is deterministic and identical on all machines.</td></tr><tr><td>Position Strength</td><td>How aggressively the rigidbody chases the target position. Acts as the natural frequency of a critically-damped spring. Higher values = tighter tracking but stiffer feel. The steady-state tracking lag is approximately velocity / strength.</td></tr><tr><td>Correction Range</td><td>The distance over which position correction ramps from zero to full strength. Larger values give softer correction, letting local collisions play out naturally before being pulled back. Smaller values give tighter snapping.</td></tr><tr><td>Rotation Strength</td><td>How aggressively the rigidbody corrects rotation errors. Works the same way as position strength but for angular correction.</td></tr><tr><td>Hard Snap Distance</td><td>If the position error exceeds this distance, the rigidbody teleports to the target instead of using forces. Acts as a safety net for large desync.</td></tr><tr><td>Hard Snap Angle</td><td>If the rotation error (degrees) exceeds this threshold, the rotation snaps instantly instead of using torque. Set to a negative value to disable rotation snapping entirely.</td></tr><tr><td>Acceptable Rotation Error</td><td>Rotation error (degrees) below which rotation correction stops, preventing micro-jitter at rest. Set to a negative value to disable rotation correction entirely.</td></tr><tr><td>Position Change Threshold</td><td>Minimum distance the rigidbody must move before triggering a network update. Prevents unnecessary updates while stationary.</td></tr><tr><td>Rotation Change Threshold</td><td>Minimum angle the rigidbody must rotate before triggering a network update.</td></tr><tr><td>Velocity Stop Threshold</td><td>If both linear and angular velocities are below this value, the object is considered stopped and will cease sending updates.</td></tr></tbody></table>

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

If you're replacing a `Rigidbody` reference with `NetworkRigidbody`, you do **not** need to change any of your existing `MovePosition`, `MoveRotation`, or `AddForce` calls — they work identically. Only use `TeleportTo` when you specifically need an instant, non-physical reposition.
