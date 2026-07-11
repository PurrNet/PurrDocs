---
description: Non-mono bound predicted code for maximum flexibility
---

# Predicted Modules

The PredictedModule system allows you to encapsulate specific game logic and state into reusable, self-contained units. Instead of writing a monolithic `PredictedIdentity` that handles movement, health, inventory, and abilities all in one script, you can break these features down into individual modules.

## Why use modules?

* Encapsulation: Keep logic and state (e.g., a Timer or Health system) isolated from other systems.
* Reusability: Write a module once (like a `ProjectileMovementModule`) and drop it into any `PredictedIdentity`.
* Network efficiency: Modules have their own delta compression, so unchanged module state does not need a full resend.
* Automatic History & Rollbacks: Modules automatically participate in the prediction rollback system, saving you from manually managing history buffers for every variable.

Modules follow the simulation and reconciliation timeline of their containing identity. Configure [prediction policies](../prediction-policies.md) on the identity or its nearest scope, not on individual modules.

{% hint style="info" %}
Performance note: A predicted module acts similar to a predicted identity. This means that it's another state, and another simulation to handle. Pros of this is that it adds flexibility and modularity easily. The con being that now your identity is handling multiple simulations and multiple states which can be heavier for performance on both CPU and bandwidth. It's about weighing re-usability and flexibility vs performance.\
However, this is **not** heavier than having multiple predicted identities.
{% endhint %}

***

## Implementation Guide

Creating a module involves two steps: defining the state and creating the module logic.

**1. Define the State**

Create a struct that implements `IPredictedData<T>`. This holds the data you want to sync and predict.

```csharp
public struct HealthState : IPredictedData<HealthState>
{
    public int currentHealth;
    public int maxHealth;

    public void Dispose() { }
}
```

**2. Create the Module**

Inherit from `PredictedModule<TState>`. You typically override `Simulate` for logic and `UpdateView` for visuals.

```csharp
public class HealthModule : PredictedModule<HealthState>
{
    // You can customize the constructor for custom needs
    public HealthModule(PredictedIdentity identity, int startingHealth) : base(identity)
    {
        currentState.currentHealth = startingHealth;

        // Updates the visual buffer to match the new current state immediately
        ResetInterpolation();
    }

    // logic: runs on fixed ticks
    protected override void Simulate(ref HealthState state, float delta)
    {
        // Example: Regen health over time
        if (state.currentHealth < state.maxHealth)
        {
            state.currentHealth++;
        }
    }

    public void ChangeHealth(int change) => currentState.currentHealth += change;

    // Visuals: runs every frame
    protected override void UpdateView(HealthState viewState, HealthState? verifiedState)
    {
        // Update UI or visual effects based on the interpolated viewState
        // Easiest to utilize events to communicate out of the module
    }
}
```

***

#### Using a Module

Modules constructed before the first simulation tick (typically inside `LateAwake`) are static. They live for the lifetime of the identity and are not state-tracked. Modules constructed during simulation are dynamic and replicate automatically.

To use a module, instantiate it within your `PredictedIdentity`. It will automatically register itself with the identity's prediction lifecycle.

```csharp
public class PlayerController : PredictedIdentity
{
    private HealthModule _health;
    private TimerModule _timer; // Built-in example

    protected override void LateAwake()
    {
        base.Awake();
        // Create and register the modules
        _health = new HealthModule(this, 100);
        _timer = new TimerModule(this);
    }

    // You can now access public methods or properties of your modules
    public void TakeDamage(int amount)
    {
        _health.ChangeHealth(-amount);
    }
}
```

***

#### Dynamic Modules

Modules can also be constructed during simulation. When this happens, the module becomes part of the replicated state. Other peers reconcile the module's existence automatically through the same rollback path used for predicted state.

This is useful when a module's existence depends on gameplay events. For example, attaching a temporary `BurnEffectModule` when a player catches fire, then disposing it when the effect ends.

```csharp
protected override void Simulate(MyInput input, ref MyState state, float delta)
{
    if (input.applyBurn && !TryGetModule<BurnEffectModule>(out _))
    {
        var burn = new BurnEffectModule(this);
        burn.StartFor(3f);
    }
}
```

Disposing a dynamic module is done by calling `Dispose()` on the module itself. The module is removed from the identity and torn down on every peer through the same reconcile path.

```csharp
protected override void Simulate(MyInput input, ref MyState state, float delta)
{
    if (input.cancelBurn && TryGetModule<BurnEffectModule>(out var burn))
        burn.Dispose();
}
```

Constraints on dynamic modules:

* The module type must expose a public constructor whose first parameter is `PredictedIdentity`. Any additional parameters must be optional. The reconcile path uses this constructor to recreate the module on remote peers.
* Per-instance configuration that affects simulation must live in the module's state, not in constructor arguments. Optional constructor parameters fall back to their defaults during reconciliation.
* Static modules cannot be disposed. Calling `Dispose()` on a module created before the first simulation tick is a no-op and logs an error.

The `onDisposed` event fires when a module is torn down, whether by an explicit `Dispose()` call or by rollback reconciliation. Use it to drop any external references.

***

#### Looking up Modules

Caching a dynamic module reference outside state is risky. Rollback may tear the module down and recreate it on replay, leaving the cached reference dangling.

Prefer looking the module up from the identity's live module list:

```csharp
if (TryGetModule<BurnEffectModule>(out var burn))
    burn.Refresh();
```

Available accessors on `PredictedIdentity`:

<table data-header-hidden><thead><tr><th width="240"></th><th></th></tr></thead><tbody><tr><td><strong>Member</strong></td><td><strong>Description</strong></td></tr><tr><td><code>modules</code></td><td>The live, ordered list of modules attached to the identity. Static first, then dynamic in registration order.</td></tr><tr><td><code>TryGetModule&#x3C;T>(out T module)</code></td><td>Returns the first module of type <code>T</code> attached to the identity.</td></tr><tr><td><code>GetModules&#x3C;T>(List&#x3C;T> results)</code></td><td>Appends every module of type <code>T</code> to the given list, in registration order.</td></tr></tbody></table>

***

#### Key Overrides

<table data-header-hidden><thead><tr><th width="177"></th><th></th></tr></thead><tbody><tr><td><strong>Method</strong></td><td><strong>Description</strong></td></tr><tr><td><code>Simulate</code></td><td>Main Logic. Executed every tick. Modify <code>state</code> here to advance simulation.</td></tr><tr><td><code>UpdateView</code></td><td>Visuals. Executed every frame. Use <code>viewState</code> (interpolated) for smooth rendering.</td></tr><tr><td><code>Initialize</code></td><td>Setup. Called when the module is created. Use this instead of Awake/Start.</td></tr><tr><td><code>Interpolate</code></td><td>Smoothing. (Optional) Custom logic for blending states between ticks. Defaults to standard linear interpolation.</td></tr></tbody></table>
