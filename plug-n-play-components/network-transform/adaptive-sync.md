# Adaptive sync

Adaptive sync reduces Network Transform bandwidth without lowering the configured tick rate. The controlling peer still captures the transform every tick, but skips an update while receiving peers can reconstruct the missing states accurately.

The sender verifies the same reconstruction against the states it captured. As soon as the reconstructed motion no longer matches, Network Transform sends an update and establishes a new baseline. Erratic movement therefore falls back to normal syncing automatically.

Adaptive sync is enabled by default and does not add render delay. The built-in strategy handles:

* Straight movement and scale changes through linear reconstruction
* Curved movement such as turns, orbits, arches and projectile-like paths
* Rotation continuing at a constant angular rate

It also forces periodic updates during reconstructible motion. The default maximum interval is `0.3` seconds, limiting how long packet loss can leave a receiver without a new baseline.

{% hint style="info" %}
Keep the built-in strategy unless your object follows a motion model it cannot reconstruct well. A custom strategy only saves more bandwidth when its reconstruction matches the object's real movement.
{% endhint %}

## Runtime control

Adaptive sync can be toggled at runtime:

```csharp
networkTransform.adaptiveSync = true;
```

`hasSyncStrategy` reports whether an adaptive strategy is currently active. Disabling `adaptiveSync` also disables any custom strategy assigned to the component.

To use the built-in strategy with a different maximum send interval, create and assign it directly:

```csharp
var strategy = new NetworkTransformDefaultStrategy
{
    maxSendInterval = 0.2f
};

networkTransform.SetSyncStrategy(strategy);
```

Call `SetSyncStrategy` before the object spawns where possible. Assigning one after spawning is supported and takes effect immediately. Passing `null` returns to the shared built-in strategy:

```csharp
networkTransform.SetSyncStrategy(null);
```

## Creating a strategy

A strategy is a plain C# class derived from `NetworkTransformSyncStrategy`. Override `TryReconstruct` to describe how a state should be reconstructed from the three states supplied to it.

This example fits a quadratic curve through the three position samples. Rotation and scale keep the linear values already provided in `result`:

```csharp
using PurrNet;
using UnityEngine;

public sealed class QuadraticPositionStrategy : NetworkTransformSyncStrategy
{
    protected override bool TryReconstruct(
        in NetworkTransformSample prev,
        in NetworkTransformSample from,
        in NetworkTransformSample to,
        float t,
        ref NetworkTransformSample result)
    {
        Vector3 velocity = (to.position - prev.position) * 0.5f;
        Vector3 acceleration = prev.position - 2f * from.position + to.position;

        result.position = from.position + velocity * t + acceleration * (0.5f * t * t);
        return true;
    }
}
```

Assign it from a component on the same object:

```csharp
using PurrNet;
using UnityEngine;

[RequireComponent(typeof(NetworkTransform))]
public sealed class QuadraticPositionSync : MonoBehaviour
{
    private void Awake()
    {
        var strategy = new QuadraticPositionStrategy
        {
            maxSendInterval = 0.2f
        };

        GetComponent<NetworkTransform>().SetSyncStrategy(strategy);
    }
}
```

{% hint style="warning" %}
Assign the same strategy with the same settings on every peer. `SetSyncStrategy` does not replicate the strategy itself. Keeping the setup component on the shared network prefab ensures the sender and receivers use the same reconstruction.
{% endhint %}

`TryReconstruct` receives world-space samples. `t` is `0` at `from`, `1` at `to`, and can be greater than `1` when extrapolating. `result` is already filled with the linear reconstruction, so only replace the properties your strategy handles. Return `true` when the result was shaped, or `false` to keep the linear reconstruction.

The method must be pure and deterministic. It runs independently on the sender and receivers, so it must not depend on `Time`, randomness, scene queries, or mutable state that is not derived from the supplied samples. A strategy instance can be shared between Network Transforms when it is stateless, or when any cached state is keyed entirely by its inputs.

The strategy does not blindly suppress updates. PurrNet compares its output with the captured transform states and resumes sending whenever the result falls outside the accepted serialized tolerance.

## Combining strategies

`NetworkTransformCompositeStrategy` tries several strategies in order. The first strategy returning `true` provides the reconstructed state. This is useful when an object has a specialized movement mode but should fall back to the built-in strategy for everything else:

```csharp
var strategy = new NetworkTransformCompositeStrategy
{
    maxSendInterval = 0.2f,
    strategies = new NetworkTransformSyncStrategy[]
    {
        new RailMovementStrategy(),
        new NetworkTransformDefaultStrategy()
    }
};

networkTransform.SetSyncStrategy(strategy);
```

The composite's `maxSendInterval` is used for the whole chain. Intervals set on its child strategies are ignored.
