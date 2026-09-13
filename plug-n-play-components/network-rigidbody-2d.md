---
icon: circle-dot
---

# Network Rigidbody 2D

This is the 2D version of the [Network Rigidbody](network-rigidbody.md). It syncs a `Rigidbody2D` over the network and uses physics to smoothly align it on other peers, so locally everything still feels responsive.

Under the hood both components share the same core, so anything you know from the 3D version applies here as well. The differences are purely the 2D API: positions are `Vector2`, rotation is a single angle in degrees and forces use `ForceMode2D`.

{% hint style="info" %}
If you need physics in multiplayer, you might want to read the [Physics In Multiplayer](../guides/physics-in-multiplayer.md) page
{% endhint %}

### Settings

The settings are the same as on the 3D component, so the defaults should work for most cases, but it's worth knowing what they do.

<table><thead><tr><th width="230">Setting</th><th>Effect</th></tr></thead><tbody><tr><td>Owner Auth</td><td>Decides who holds the truth of the simulation. If true, the owning client calculates physics (Client Auth). If false, the server does (Server Auth).</td></tr><tr><td>Space</td><td>Local syncs relative to the current parent, World syncs in world space.</td></tr><tr><td>Sync Parent</td><td>Syncs parent changes through the hierarchy. Only works when the new parent has a Network Identity.</td></tr><tr><td>Use Parent Frame Position Error</td><td>Measures correction and hard-snap distance in the active parent or soft-parent frame. This prevents parent motion by itself from appearing as child position error.</td></tr><tr><td>Settings Override</td><td>Optional <code>NetworkRigidbody2DSettings</code> asset that takes over all correction decisions. See below.</td></tr><tr><td>Interpolation Delay</td><td>How far behind real-time (in seconds) the interpolation target sits. Higher values absorb more network jitter but add visual latency. Default: 0.05s.</td></tr><tr><td>Auto Interpolation Delay</td><td>Floors the interpolation delay at one send interval, since below that there is never a newer snapshot to interpolate towards.</td></tr><tr><td>Prediction Factor</td><td>Pushes the target position forward using velocity to compensate for interpolation delay. 0 = no prediction, 1 = compensate for the full interpolation delay, >1 = predict further ahead. The offset is identical on all machines.</td></tr><tr><td>Position Strength</td><td>How aggressively the rigidbody chases the target position. Acts as the natural frequency of a critically-damped spring. Higher values = tighter tracking but stiffer feel.</td></tr><tr><td>Correction Range</td><td>The distance over which position correction ramps from zero to full strength. Larger values give softer correction, letting local collisions play out before being pulled back.</td></tr><tr><td>Rotation Correction</td><td>How rotation is corrected on observers. <code>Torque</code> uses a torque spring, <code>Kinematic</code> follows the target rotation exactly with <code>MoveRotation</code>. <code>Auto</code> picks Kinematic when the Rigidbody2D has Freeze Rotation enabled (typical for platformer characters) and Torque otherwise.</td></tr><tr><td>Rotation Strength</td><td>How aggressively the rigidbody corrects rotation errors. Only used by the Torque mode.</td></tr><tr><td>Hard Snap Distance</td><td>If the position error exceeds this distance, the rigidbody teleports to the target instead of using forces. Acts as a safety net for large desync.</td></tr><tr><td>Hard Snap Angle</td><td>If the rotation error (degrees) exceeds this threshold, the rotation snaps instantly instead of using torque. Set to a negative value to disable rotation snapping entirely.</td></tr><tr><td>Acceptable Rotation Error</td><td>Rotation error (degrees) below which rotation correction stops, preventing micro-jitter at rest. Set to a negative value to disable rotation correction entirely.</td></tr><tr><td>Stall Recovery Delay</td><td>Master switch for stall recovery: eases a settled body onto the settled target and recovers rotation errors torque can't close. Negative to disable.</td></tr><tr><td>Position Change Threshold</td><td>Minimum distance the rigidbody must move before triggering a network update. Prevents unnecessary updates while stationary.</td></tr><tr><td>Rotation Change Threshold</td><td>Minimum angle the rigidbody must rotate before triggering a network update.</td></tr><tr><td>Velocity Stop Threshold</td><td>If both linear and angular velocities are below this value, the object is considered stopped and will cease sending updates.</td></tr></tbody></table>

### Using it from code

The component mirrors the `Rigidbody2D` API, so if you're replacing a `Rigidbody2D` reference with `NetworkRigidbody2D`, your existing `AddForce`, `MovePosition` and `MoveRotation` calls work as they are.

```csharp
[SerializeField] private NetworkRigidbody2D _rb;

private void FixedUpdate()
{
    if (!_rb.isController)
        return;

    _rb.AddForce(Vector2.right * moveInput * moveForce);

    if (jumpPressed)
        _rb.AddForce(Vector2.up * jumpForce, ForceMode2D.Impulse);
}
```

Forces called from a non-controller are forwarded to the controller, so things like a server-side explosion pushing an owner-auth player still work.

The synced Rigidbody2D settings (`mass`, `drag`, `angularDrag`, `gravityScale` and `bodyType`) are exposed as properties too. Changing them through the component sends them to the other peers, changing them directly on the `Rigidbody2D` does not.

```csharp
_rb.gravityScale = 0f;
_rb.bodyType = RigidbodyType2D.Kinematic;
```

{% hint style="warning" %}
Only `Dynamic` and `Kinematic` body types are distinguished on the wire. A `Static` body on the controller shows up as `Kinematic` on observers.
{% endhint %}

### Parent-relative correction

Just like the 3D version, a Rigidbody2D on a moving platform can be synced relative to its parent. Velocity is always relative to the parent's motion when a parent frame is active, and you can opt in to measuring position error in the parent frame as well:

```csharp
_rb.useParentFramePositionError = true;
```

This works with a real parent and with `SetSoftParent`. The parent's velocity contribution comes from a `Rigidbody2D` on the parent or one of its ancestors; without one it's treated as zero.

A soft parent uses another Network Identity as the sync frame without changing the Unity transform hierarchy. Call it on the controlling peer:

```csharp
_rb.SetSoftParent(movingPlatformIdentity);

// Return to the frame selected by the Space setting and real Unity parent.
_rb.ClearSoftParent();
```

### Teleporting

Regular physics operations (`position`, `rotation`, `MovePosition`, `MoveRotation`, `AddForce`, etc.) are synced automatically and should be used for all regular gameplay movement. For instant repositioning like respawns or portals, use `TeleportTo`:

```csharp
// Teleport to a new position and rotation (degrees)
_rb.TeleportTo(spawnPoint.position, spawnPoint.eulerAngles.z);

// Teleport to just a new position (keeps current rotation)
_rb.TeleportTo(respawnPosition);
```

`TeleportTo` zeros out velocity, clears the interpolation buffer on all observers and sends a reliable RPC so the teleport is never missed. If you only want to fix up the local peer without sending anything, use `TeleportLocal` instead.

To react when the correction performs a hard position teleport because the error exceeded the threshold, subscribe to `onTeleportCorrection`:

```csharp
private void OnEnable()
{
    _rb.onTeleportCorrection += OnTeleportCorrection;
}

private void OnDisable()
{
    _rb.onTeleportCorrection -= OnTeleportCorrection;
}

private void OnTeleportCorrection(Rigidbody2DCorrectionContext context)
{
    Debug.Log($"Corrected from {context.previousPosition} to {context.targetPosition}");
}
```

The context is the 2D counterpart of `RigidbodyCorrectionContext`: positions are `Vector2`, rotations are in degrees and angular velocity is in degrees per second, matching `Rigidbody2D`.

### Settings override

If the built-in correction doesn't fit your game, you can take over the decisions with a `NetworkRigidbody2DSettings` asset. Create a ScriptableObject deriving from `NetworkRigidbody2DSettings<T>` and return your own `NetworkRigidbody2DSettingsInstance` from `CreateTyped`. Every virtual has a default that matches the built-in behaviour, so you only override what you need:

```csharp
public class TopDownCarSettings : NetworkRigidbody2DSettings<TopDownCarSettings.Instance>
{
    protected override Instance CreateTyped(NetworkRigidbody2D networkRigidbody) => new Instance();

    public class Instance : NetworkRigidbody2DSettingsInstance
    {
        public override bool ShouldTeleport(in Rigidbody2DCorrectionContext ctx)
        {
            // Never teleport while the car is sliding fast, let the spring catch up instead.
            return ctx.targetLinearVelocity.magnitude < 2f && ctx.positionError >= ctx.hardSnapDistance;
        }

        public override void ApplyRotationCorrection(in Rigidbody2DCorrectionContext ctx)
        {
            ApplyRotationSpring(ctx.rigidbody, ctx.targetRotation, ctx.targetAngularVelocity, ctx.rotationStrength * 0.5f);
        }
    }
}
```

Assign the asset to the `Settings Override` field and the inspector values are passed to your instance through the context as defaults.

### Position transforms (origin-shifted / large worlds)

Position transforms work exactly as described on the [Network Rigidbody](network-rigidbody.md#position-transforms-origin-shifted-large-worlds) page. `INetworkRigidbodyPositionTransform` takes a `NetworkRigidbodyBase`, which both the 2D and 3D component derive from, so the same implementation serves both.

```csharp
// Applies to every NetworkRigidbody and NetworkRigidbody2D in the process.
NetworkRigidbodyBase.defaultPositionTransform = new OriginShiftTransform();
```
