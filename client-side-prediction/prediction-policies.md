# Prediction Policies

A prediction policy controls how one `PredictedIdentity` participates in client simulation and reconciliation. Policies do not change server authority: the server always simulates every identity normally.

## Choosing a policy

| Policy | Client behavior | Good fit |
| --- | --- | --- |
| `FullPrediction` | Simulates every local tick, restores verified state, then replays newer ticks. | Local players, gameplay-critical physics, combat state, and objects that must interact accurately with the predicted world. |
| `ServerRelay` | Runs only verified server ticks on clients and never predicts into the local future. Relay rigidbodies are kinematic on clients and act as verified geometry for predicted objects. | Remote actors, server-driven hazards, or objects where latency is acceptable and rollback cost is not. |
| `SoftCorrection` | Keeps simulating locally, skips live rollback, compares verified state with client history, and blends the error into the current simulation. | Debris, props, ragdolls, and other physics where convergence matters more than an exact historical pose. |
| `PredictedIfOwned` | Resolves to `FullPrediction` for the local owner and `ServerRelay` for everyone else, including unowned copies. | Player pawns and vehicles that should feel immediate only to their controller. |

`ServerRelay` intentionally lives behind the local predicted timeline by roughly network latency plus buffering. `SoftCorrection` stays responsive but is convergent rather than historically exact, so it is a poor choice for gameplay-critical collision or hit validation.

{% hint style="warning" %}
`SoftCorrection` is supported only by identities that implement verified-state correction. The built-in `PredictedTransform`, `PredictedRigidbody`, `PredictedRigidbody2D`, and `PredictedProjectile3D` support it. Unsupported and deterministic identities normalize it to `FullPrediction` and report an error when appropriate.
{% endhint %}

When a Rigidbody or projectile shares a GameObject with `PredictedTransform`, the physics component controls the transform's effective policy so pose and velocity follow the same timeline.

## Configure one identity

Every Predicted Identity inspector has two policy fields:

* **Prediction Policy Source** uses the nearest active scope or overrides scopes for this identity.
* **Prediction Policy** is the local configured policy. It is also the fallback when no scope exists.

Use an override for an exceptional object. Use a scope when a prefab hierarchy or group of objects shares the same policy.

## Policy scopes

Add **PurrDiction > Prediction Policy Scope** to a GameObject. Descendant identities using scopes resolve the nearest active scope, including one on their own GameObject.

A child scope can enable **Inherit From Parent**. It then resolves the nearest active scope above it instead of its own configured value. Reparenting an identity, enabling or disabling a scope, or changing a scope's policy refreshes affected identities.

```csharp
public PredictionPolicyScope remoteActors;

private void Awake()
{
    remoteActors.SetPredictionPolicy(PredictionPolicy.ServerRelay);
}
```

## Runtime switching

There are three deliberately different APIs:

```csharp
// Change the applied policy only. A later scope refresh can replace it.
identity.SetPredictionPolicy(PredictionPolicy.ServerRelay);

// Configure a persistent per-identity override.
identity.SetPredictionPolicyOverride(PredictionPolicy.SoftCorrection);

// Remove the override and resolve the nearest active scope again.
identity.UsePredictionPolicyScope();
```

`configuredPredictionPolicy` changes the serialized/local fallback policy. `predictionPolicy` is the currently applied result, and `GetResolvedPredictionPolicy()` returns the active or currently resolvable value.

Policy changes are safest at ownership or gameplay phase boundaries. The next reconciliation aligns the identity with its new timeline. Persistent overrides and scope configuration are restored correctly when pooled objects are reused.

## Soft-correction tuning

Built-in pose components expose:

* **Correction Rate**: exponential rate used to consume the remaining error each second.
* **Snap Position Threshold**: remaining position error that applies immediately instead of blending.
* **Snap Rotation Threshold**: equivalent angular threshold in degrees.
* **Soft Velocity Correction Rate** on Rigidbody and projectile components: rate used to blend verified velocity error.

Start with the defaults, then tune while testing real latency and packet loss. A high rate converges faster but looks more like a snap; a low rate looks smoother but leaves the client divergent for longer.
