# Predicted Rigidbody (2D and 3D)

`PredictedRigidbody` and `PredictedRigidbody2D` capture Unity body state into prediction history, restore it for reconciliation, and expose familiar force and movement APIs for tick simulation.

{% hint style="warning" %}
Unity physics is not guaranteed to produce bit-identical results across machines. These components make it rollback-compatible; authoritative frames and reconciliation still correct divergence. Use deterministic identities and deterministic math when exact cross-platform simulation is required.
{% endhint %}

## Setup

For 3D:

1. Add a `Rigidbody`, `PredictedTransform`, and `PredictedRigidbody` to the same GameObject.
2. Enable **Physics 3D** under the Prediction Manager's Built In Systems.
3. Enable **Unity Physics 3D** in Physics Provider.

For 2D, use `Rigidbody2D` and `PredictedRigidbody2D`, then enable the corresponding 2D settings.

PurrDiction advances enabled physics scenes during its tick. Put gameplay forces and movement in `Simulate`, not `FixedUpdate`:

```csharp
protected override void Simulate(PlayerInput input, ref PlayerState state, float delta)
{
    predictedRigidbody.AddForce(input.move * acceleration, ForceMode.Acceleration);

    if (input.jump)
        predictedRigidbody.AddForce(Vector3.up * jumpImpulse, ForceMode.Impulse);
}
```

The 3D wrapper exposes `AddForce`, `AddTorque`, relative-force variants, explosion force, `MovePosition`, `MoveRotation`, pose and velocity properties, gravity, and kinematic state. The 2D wrapper provides the equivalent Rigidbody2D operations.

## Settings and policies

`PredictedRigidbody` exposes **Float Accuracy** for packed 3D body state and an **Event Mask** for selecting collision and trigger callbacks. Both body types expose **Soft Velocity Correction Rate** for `SoftCorrection`.

Policy behavior is specialized for physics:

* `FullPrediction` simulates the body and rewinds it during replay.
* `ServerRelay` keeps the body kinematic on clients and poses it from verified state.
* `SoftCorrection` freezes the body during replay, then blends verified pose and velocity error into the live body.
* `PredictedIfOwned` uses full prediction for the local owner and server relay elsewhere.

See [Prediction Policies](../prediction-policies.md) before mixing different policy types in the same collision scene.

## Collision and trigger events

Subscribe to the component's prediction-aware events instead of treating ordinary Unity callbacks as final gameplay notifications:

```csharp
private void OnEnable()
{
    predictedRigidbody.onCollisionEnter += OnPredictedCollision;
}

private void OnDisable()
{
    predictedRigidbody.onCollisionEnter -= OnPredictedCollision;
}

private void OnPredictedCollision(GameObject other, PhysicsCollision collision)
{
    // Mutate predicted state here. Gate one-shot VFX/audio separately.
}
```

`PhysicsCollision` contains predicted contact data, relative velocity, and impulse. The 2D event supplies a `DisposableList<Physics2DContactPoint>` that is owned by PurrDiction; consume it during the callback and do not retain it.

For one-shot presentation, use `PredictedEvent`, `predictionManager.isVerifiedView`, or defer the effect to view code so catch-up and replay do not duplicate it.
