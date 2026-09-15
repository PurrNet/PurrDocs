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

Torque follows Unity's own semantics. `AddTorque` and `AddRelativeTorque` on the 3D wrapper apply the body's world-space inverse inertia tensor for `Force` and `Impulse` modes; locked or zero inertia axes receive no angular velocity, and `Acceleration` or `VelocityChange` bypass mass and inertia. The 2D wrapper divides by the body's moment of inertia and works in degrees per second. The 3D relative variants rotate the vector by the body's rotation only, so object scale does not distort applied forces, and `AddForceAtPosition` derives its torque from the lever arm about the center of mass in both 2D and 3D.

Always drive the body through the wrapper inside `Simulate`, never through the raw Unity `Rigidbody`. On clients where the resolved policy is `ServerRelay`, the body is forced kinematic and posed from verified state; the 3D wrapper's velocity setters, which every force method routes through, silently do nothing on a kinematic body, and the 2D wrapper skips static bodies. The same `Simulate` code therefore runs on every peer without fighting the relayed pose; writing to the raw component bypasses these guards.

## Settings and policies

`PredictedRigidbody` exposes **Float Accuracy** for packed 3D body state and an **Event Mask** for selecting collision and trigger callbacks. Both body types expose **Soft Velocity Correction Rate** for `SoftCorrection`.

{% version range=">=1.4.0" %}

The default **Event Mask** is Collision Enter, Collision Exit, Trigger Enter, and Trigger Exit. The Stay variants are opt-in because Unity reports them once per touching pair per physics step, which adds up fast on a busy body. Turn them on only where you actually read them.

Behind the scenes, the Unity collision messages live on small hidden proxy components that are attached only while the matching event kinds are enabled. Unity generates contact reports and a managed callback for every script class that declares a collision message, whether or not that message does anything, so a body with no enabled events pays nothing. The proxies are hidden in the inspector and never serialized.

You can change the mask at runtime through the `eventMask` property. Setting it attaches or detaches the proxies right away:

```csharp
// Only listen for trigger stay while the ability is active.
predictedRigidbody.eventMask = PredictedRigidbody.DEFAULT_EVENT_MASK | PhysicsEventMask.TriggerStay;
```

{% endversion %}

Policy behavior is specialized for physics:

* `FullPrediction` simulates the body and rewinds it during replay.
* `ServerRelay` keeps the body kinematic on clients and poses it from verified state.
* `SoftCorrection` freezes the body during replay, then blends verified pose and velocity error into the live body.
* `PredictedIfOwned` uses full prediction for the local owner and server relay elsewhere.
* `PredictedIfOwnedWithSoftFallback` uses full prediction for the local owner and soft correction elsewhere, so remote bodies keep simulating and converge without rollback.

See [Prediction Policies](../prediction-policies.md) before mixing different policy types in the same collision scene.

## Collision and trigger events

Subscribe to the component's prediction-aware events instead of treating ordinary Unity callbacks as final gameplay notifications.

{% version range="<1.4.0" %}

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

{% endversion %}

{% version range=">=1.4.0" %}

The events are `onPredictedCollisionEnter`, `onPredictedCollisionExit`, `onPredictedCollisionStay`, `onPredictedTriggerEnter`, `onPredictedTriggerExit`, and `onPredictedTriggerStay`. Each one hands you a single struct:

```csharp
private void OnEnable()
{
    predictedRigidbody.onPredictedCollisionEnter += OnPredictedCollision;
    predictedRigidbody.onPredictedTriggerExit += OnPredictedTriggerExit;
}

private void OnDisable()
{
    predictedRigidbody.onPredictedCollisionEnter -= OnPredictedCollision;
    predictedRigidbody.onPredictedTriggerExit -= OnPredictedTriggerExit;
}

private void OnPredictedCollision(PredictedCollision collision)
{
    // collision.other      -> the other GameObject
    // collision.otherId    -> its PredictedComponentID
    // collision.collision  -> PhysicsCollision with contacts, relative velocity, and impulse
}

private void OnPredictedTriggerExit(PredictedTrigger trigger)
{
    // trigger.other is null when the other object was deleted this tick.
    // trigger.otherId is still valid, so bookkeeping keyed by id keeps working.
    currentState.overlapping.Remove(trigger.otherId);
}
```

| Payload | Fields |
| --- | --- |
| `PredictedTrigger` | `other`, `otherId` |
| `PredictedCollision` (3D) | `other`, `otherId`, `collision` (`PhysicsCollision`) |
| `PredictedCollision2D` (2D) | `other`, `otherId`, `contacts` (`DisposableList<Physics2DContactPoint>`) |

The id is the whole reason these exist. When a predicted object is deleted from the hierarchy while it is touching something, the Exit event still fires, but the GameObject is already gone by then, so `other` is null. `otherId` is always the id the other object had when the contact was recorded, which makes it safe to key overlap sets, damage tables, or cooldowns by id and clean them up on Exit. Do not store the GameObject in state; store the `PredictedComponentID` and resolve it through the hierarchy when you need the object.

The 2D `contacts` list is owned by PurrDiction; consume it during the callback and do not retain it.

The older `onCollisionEnter`, `onCollisionExit`, `onCollisionStay`, `onTriggerEnter`, `onTriggerExit`, and `onTriggerStay` events still fire, but they are marked obsolete. They pass only the GameObject, so an Exit caused by the other object being deleted is skipped for them entirely; only the struct events see it. Migrate by swapping the event name and changing the handler signature to take the struct.

{% endversion %}

For one-shot presentation, use `PredictedEvent`, `predictionManager.isVerifiedView`, or defer the effect to view code so catch-up and replay do not duplicate it.

For collision and trigger events on predicted objects that are not rigidbodies, such as a Character Controller, see [Predicted Physics Callbacks](predicted-physics-callbacks.md).
