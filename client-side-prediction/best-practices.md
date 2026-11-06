# Best Practices

Prediction failures usually come from state that cannot be restored, simulation code with hidden side effects, or a client timeline that does not match the object's gameplay role.

## Put every mutable simulation value in state

`INPUT` and `STATE` must be structs implementing the appropriate `IPredictedData` interface. State should include every value that can change a later simulation result: position, velocity, cooldowns, counters, targets, random state, active modes, and stable predicted object IDs.

```csharp
public struct PlayerInput : IPredictedData
{
    public Vector2 move;
    public bool jump;
    public void Dispose() { }
}

public struct PlayerState : IPredictedData<PlayerState>
{
    public Vector3 position;
    public Vector3 velocity;
    public float jumpCooldown;
    public void Dispose() { }
}
```

Inspector fields are configuration, not mutable simulation state. If a serialized value can change at runtime and affect gameplay, copy it into state or change it through a predicted operation.

## Initialize and bridge state deliberately

Use `GetInitialState` for a complete starting snapshot. Use `GetUnityState` and `SetUnityState` when state lives partly in Unity components such as Transform or Rigidbody.

Do not change a Unity component during simulation without ensuring the corresponding state captures and restores that change. Built-in transform and physics components already implement this bridge.

## Keep simulation replay-safe

`Simulate` and `LateSimulate` can execute more than once for the same tick. They may mutate predicted state, create objects through `PredictedHierarchy`, and use deterministic built-in systems. They must not directly perform irreversible work such as:

* Playing audio or particles
* Sending analytics or achievements
* Writing save data
* Calling external services
* Sending ordinary RPCs on every replay

Use `PredictedEvent`, `predictionManager.isVerifiedView`, or view callbacks for one-shot presentation. Do not skip deterministic state mutations merely because `isCatchingUpFrames` is true.

## Don't mix regular networking into predicted gameplay

A `PredictedIdentity` is not a `NetworkIdentity`. Predicted gameplay is synced by input, state and reconciliation, and the usual PurrNet tools either don't work on it or fight with rollback. Inside predicted gameplay, use the predicted equivalent:

| Instead of | Use |
| --- | --- |
| `ServerRpc`, `ObserversRpc`, `TargetRpc` to change gameplay | Input for player intent, state for the result. PurrNet's codegen rejects a non-static RPC on a class that is not a `NetworkIdentity` or `NetworkClass`, and an RPC sent from `Simulate` goes out again on every replay. |
| `SyncVar` and other sync types | Fields in your predicted state. Sync types only sync through a spawned `NetworkIdentity`. |
| `Instantiate` and `Destroy` | `predictionManager.hierarchy.Create` and `Delete` (see [Predicted Hierarchy](predicted-hierarchy.md)). |
| `NetworkTransform` | `PredictedTransform`, or `PredictedRigidbody` / `PredictedRigidbody2D` for physics. Two components writing the same Transform will fight. |
| `NetworkAnimator` | Set `Animator` parameters from the view state in `UpdateView` (see [Views and Interpolation](views-and-interpolation.md)). |
| A `Transform`, `GameObject` or `PredictedIdentity` reference in state, input or an RPC | A `PredictedObjectID` or `PredictedComponentID`, resolved back when needed (see [Predicted Object References](predicted-object-references.md)). Instances can be recreated or pooled across rollbacks, so a direct reference can point at the wrong object or nothing. PurrNet also can't pack a Transform that is not under a spawned `NetworkIdentity`; it sends it as null and logs a `packed as null` warning. |

This also applies to the component returned by `TryCreateAndGet`: use it right away in the same tick, and store its ID if you need it later.

You can still use RPCs and sync types for things outside the simulation, such as chat, lobby, menus, or a display name. If a predicted prefab needs them, put a `NetworkIdentity` on it; the prediction system spawns and despawns it with the predicted object (see [Network Identities in Predicted Prefabs](built-in-components/predicted-identity-spawner.md)). Keep gameplay decisions in predicted state.

## Read time and randomness from the predicted world

Do not use `Time.deltaTime`, `Time.time`, `UnityEngine.Random`, frame count, or wall-clock time in tick simulation. Use the callback's `delta`, `PredictedTime`, and `PredictedRandom` so rollback recreates the same sequence.

For strict cross-platform determinism, use `DeterministicIdentity`, `sfloat`, or fixed-point types. Unity physics is rollback-compatible through the built-in components but not guaranteed bit-identical across machines.

## Validate input on the server

Implement `SanitizeInput` to clamp ranges and remove impossible values. Validate resulting actions against authoritative state as well: a sanitized fire button still needs a server-side cooldown, ammo check, and ownership check.

Remove one-shot edges such as jump or fire from extrapolated input in `ModifyExtrapolatedInput`; otherwise a missing packet can repeat them.

## Choose the cheapest correct policy

* Use `FullPrediction` when the object affects local gameplay and must reconcile exact interactions.
* Use `PredictedIfOwned` for locally controlled actors when remote copies can follow verified state.
* Use `ServerRelay` for server-driven objects where client latency is acceptable.
* Use `SoftCorrection` only for supported, non-critical objects that may converge over time.

Avoid using soft correction merely to hide a broken deterministic simulation. It changes consistency guarantees, not just visuals.

## Treat history data as owned memory

PurrDiction copies and disposes states frequently. Use PurrNet disposable collections for variable-sized state, implement `Dispose`, and implement `IDuplicate<T>` when a state contains owned buffers that need deep copying. Never retain a disposable collision contact list beyond its callback.

Implement `IEquatable<T>` for complex input or state when equality checks would otherwise be expensive or ambiguous.

## Use stable predicted references

Store `PredictedObjectID` in state instead of direct GameObject references for predicted objects that can be created, deleted, pooled, or recreated during rollback. Resolve the object through `PredictedHierarchy` when needed.

## Test the failure modes

Test as a remote client, not only as host. Exercise:

* Latency, jitter, and packet loss
* Ownership transfer and reconnect
* Spawn/despawn during rollback
* Pool reuse under every configured policy
* Different render frame rates and tick rates
* Large corrections, teleports, and late joiners

Use the bandwidth and prediction profilers to verify history size, correction frequency, and per-tick state cost before optimizing individual fields.
