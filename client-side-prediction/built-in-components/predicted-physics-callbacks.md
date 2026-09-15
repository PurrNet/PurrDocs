# Predicted Physics Callbacks

`PredictedPhysicsCallbacks` gives any predicted object rollback-aware 3D collision and trigger events without needing a `PredictedRigidbody`. Use it on kinematic colliders, trigger volumes, and Character Controllers that still need to know what they touched on the simulated timeline.

It is a `StatelessPredictedIdentity`, so it carries no state of its own. Unity's collision messages are recorded during the tick, stored in the prediction event history, and raised again in the right order when the tick is replayed, exactly like the rigidbody events.

## Setup

1. Add a Collider (or Character Controller) to the GameObject.
2. Add `PredictedPhysicsCallbacks` to the same GameObject.
3. Enable **Physics 3D** under the Prediction Manager's Built In Systems and **Unity Physics 3D** in Physics Provider.

Use the **Event Mask** to pick which callbacks are recorded. Every kind is enabled by default on this component, including the Stay variants; trim it down to what you read.

{% version range=">=1.4.0" %}

## Events

The struct events match the rigidbody components: `onPredictedCollisionEnter`, `onPredictedCollisionExit`, `onPredictedCollisionStay`, `onPredictedTriggerEnter`, `onPredictedTriggerExit`, and `onPredictedTriggerStay`. See [Predicted Rigidbody](predicted-rigidbody-2d-and-3d.md#collision-and-trigger-events) for the payload structs and the `otherId` contract.

```csharp
[SerializeField] private PredictedPhysicsCallbacks callbacks;

private void OnEnable()
{
    callbacks.onPredictedTriggerEnter += OnTriggerEntered;
}

private void OnDisable()
{
    callbacks.onPredictedTriggerEnter -= OnTriggerEntered;
}

private void OnTriggerEntered(PredictedTrigger trigger)
{
    // trigger.other and trigger.otherId identify who walked in
}
```

The GameObject-only `onCollisionEnter`, `onTriggerEnter`, and friends are still raised but marked obsolete, and they skip the Exit raised by a deletion since they have no GameObject to pass.

## Character Controller hits

When the GameObject has a `CharacterController`, Unity's `OnControllerColliderHit` is recorded too. Enable **Controller Collider Hit** in the event mask and subscribe to `onControllerColliderHit`:

```csharp
callbacks.onControllerColliderHit += (GameObject other, PhysicsControllerHit hit) =>
{
    // hit.point, hit.normal, hit.moveDirection, hit.moveLength
};
```

`PhysicsControllerHit` is a packed copy of the parts of `ControllerColliderHit` that matter for gameplay, so it survives rollback and replay. It is how a predicted character controller can push rigidbodies, detect walls, or stick to moving platforms on the simulated timeline.

{% endversion %}

{% version range="<1.4.0" %}

## Events

The component exposes `onCollisionEnter`, `onCollisionExit`, `onCollisionStay`, `onTriggerEnter`, `onTriggerExit`, and `onTriggerStay`. Collision events include a `PhysicsCollision`; trigger events provide the other GameObject.

{% endversion %}

As with every predicted event, mutate predicted state inside the handler and keep one-shot presentation behind `PredictedEvent` or `predictionManager.isVerifiedView`.
