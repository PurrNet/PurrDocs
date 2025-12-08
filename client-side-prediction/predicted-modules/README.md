---
description: Non-mono bound predicted code for maximum flexibility
---

# Predicted Modules

{% hint style="warning" %}
This is new functionality and currently only available on the dev branch. Once it's been tested, it'll be released fully.

This was introduced in 1.2.2-beta.4\
In case you don't see the functionality, ensure you are at least on this version of PurrDiction
{% endhint %}

The PredictedModule system allows you to encapsulate specific game logic and state into reusable, self-contained units. Instead of writing a `PredictedIdentity` that handles singular logic like timers, inventory, health, and such all in one script, you can break these features down into individual modules.

#### Why use Modules?

* Encapsulation: Keep logic and state (e.g., a Timer or Health system) isolated from other systems.
* Reusability: Write a module once (like a `ProjectileMovementModule`) and drop it into any `PredictedIdentity`.
* Network Efficiency: Modules have their own delta compression. If only one module changes, only that module's data is sent over the network.
* Automatic History & Rollbacks: Modules automatically participate in the prediction rollback system, saving you from manually managing history buffers for every variable.

***

## Predicted Modules

The PredictedModule system allows you to encapsulate specific game logic and state into reusable, self-contained units. Instead of writing a monolithic `PredictedIdentity` that handles movement, health, inventory, and abilities all in one script, you can break these features down into individual modules.

#### Why use Modules?

* Encapsulation: Keep logic and state (e.g., a Timer or Health system) isolated from other systems.
* Reusability: Write a module once (like a `ProjectileMovementModule`) and drop it into any `PredictedIdentity`.
* Automatic History & Rollbacks: Modules automatically participate in the prediction rollback system, saving you from manually managing history buffers for every variable.

{% hint style="info" %}
Performance note: A predicted module acts similar to a predicted identity. This means that it's another state, and another simulation to handle. Pros of this is that it adds flexibility and modularity easily. The con being that now your identity is handling multiple simulations and multiple states which can be heavier for performance on both CPU and bandwidth. It's about weighing re-usability and flexibility vs performance.\
However, this is **not** heavier than having multiple predicted identities.
{% endhint %}

***

#### Implementation Guide

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

#### Key Overrides

<table data-header-hidden><thead><tr><th width="177"></th><th></th></tr></thead><tbody><tr><td><strong>Method</strong></td><td><strong>Description</strong></td></tr><tr><td><code>Simulate</code></td><td>Main Logic. Executed every tick. Modify <code>state</code> here to advance simulation.</td></tr><tr><td><code>UpdateView</code></td><td>Visuals. Executed every frame. Use <code>viewState</code> (interpolated) for smooth rendering.</td></tr><tr><td><code>Initialize</code></td><td>Setup. Called when the module is created. Use this instead of Awake/Start.</td></tr><tr><td><code>Interpolate</code></td><td>Smoothing. (Optional) Custom logic for blending states between ticks. Defaults to standard linear interpolation.</td></tr></tbody></table>
