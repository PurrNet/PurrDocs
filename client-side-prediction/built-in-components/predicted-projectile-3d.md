# Predicted Projectile 3D

`PredictedProjectile3D` is a lightweight spherical projectile simulation. It advances velocity, applies optional gravity, and sphere-casts along each tick's travel distance so fast projectiles are less likely to tunnel through geometry.

It requires `PredictedTransform`, but not a Rigidbody or Collider.

## Settings

| Setting | Effect |
| --- | --- |
| Gravity | Y acceleration applied every tick. Use `0` for straight flight. |
| Physics Material | Supplies bounciness for solid impacts. Bounciness is read during startup and is not intended to change at runtime. |
| Radius | Radius used by collision casts. It is clamped to a small positive value. |
| Is Trigger | Passes through hits and raises trigger events instead of applying a solid bounce. |
| Layer Mask | Limits collision queries to relevant layers. |
| Event Mask | Chooses which enter, exit, stay, collision, and trigger events are produced. |
| Soft Velocity Correction Rate | Blends verified velocity error when using `SoftCorrection`. |

Set initial values in the inspector, then use the runtime properties or `AddImpulse` from simulation code:

```csharp
protected override void SimulationStart()
{
    projectile.velocity = transform.forward * launchSpeed;
}

protected override void Simulate(ref ShotState state, float delta)
{
    if (state.boostThisTick)
        projectile.AddImpulse(transform.forward * boostImpulse);
}
```

## Events

The component exposes `onCollisionEnter/Exit/Stay` and `onTriggerEnter/Exit/Stay`. Solid events include `PhysicsCollision`; trigger events provide the other GameObject.

Only objects resolvable to predicted component IDs participate in the predicted event stream. Configure the Layer Mask narrowly, and use the Event Mask to avoid recording callbacks the projectile does not need.

{% hint style="info" %}
Use a Rigidbody projectile when you need full Unity physics behavior. Use Predicted Projectile 3D for high-volume, sphere-shaped shots where cast-based movement, simple gravity, bounce, and trigger handling are enough.
{% endhint %}

`PredictedProjectile3D` supports all four [prediction policies](../prediction-policies.md), including soft pose and velocity correction.

<figure><img src="../../.gitbook/assets/image (4).png" alt="Predicted Projectile 3D inspector"><figcaption></figcaption></figure>
