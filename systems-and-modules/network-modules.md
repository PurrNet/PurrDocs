# Network Modules

NetworkModule is a base class in the PurrNet networking solution that allows you to create modular, network-aware components that are not MonoBehaviours. These modules can be attached to any [NetworkIdentity](network-identity/), enabling code reuse and clean separation of concerns in your networked game.

### Key Features

* Modularity: Encapsulate specific functionality into reusable modules.
* Network Integration: Access networking properties like isServer, isClient, and send RPCs.
* Lifecycle Hooks: Override OnSpawn and OnDespawned for initialization and cleanup and many others.

### Example: HealthModule with State Synchronization

Here's a concise example of a HealthModule that synchronizes a health value across the network, ensuring proper state synchronization even if clients reconnect.

```csharp
using PurrNet;
using PurrNet.Transports;
using System;

public class HealthModule : NetworkModule
{
    private float _health;

    public event Action<float> OnHealthChanged;

    public float Health
    {
        get => _health;
        set
        {
            if (!isServer) return;

            // Synchronize health with all clients
            RpcUpdateHealth(value);
        }
    }

    public HealthModule(float initialHealth)
    {
        _health = initialHealth;
    }

    public override void OnSpawn()
    {
        base.OnSpawn();
        if (isServer)
        {
            // Ensure new or reconnecting clients receive the current health
            RpcUpdateHealth(_health);
        }
    }

    [ObserversRPC(Channel.Reliable, bufferLast: true, runLocally: true)]
    private void RpcUpdateHealth(float healthValue)
    {
        _health = healthValue;
        OnHealthChanged?.Invoke(_health);
    }
}
```

### Explanation

* Buffered RPC: The RpcUpdateHealth method uses bufferLast: true to buffer the last health value so new or reconnecting clients receive the correct state.
* Server Authority: Only the server can modify the Health property to maintain authoritative game state.
* Event Handling: Clients can subscribe to OnHealthChanged to react to health updates.

### Attaching HealthModule to a NetworkIdentity

```csharp
using UnityEngine;
using PurrNet;

public class Player : NetworkIdentity
{
    private HealthModule healthModule = new HealthModule(100); //Default to 100hp

    public override void OnSpawn()
    {
        healthModule.OnHealthChanged += OnHealthChanged;
    }

    public override void OnDespawned()
    {
        healthModule.OnHealthChanged -= OnHealthChanged;
    }

    private void OnHealthChanged(float newHealth)
    {
        // Update UI or other client-side logic
        Debug.Log($"Player health is now {newHealth}");
    }

    public void TakeDamage(float damage)
    {
        if (!isServer) return;

        healthModule.Health -= damage;

        if (healthModule.Health <= 0)
        {
            // Handle player death
            Debug.Log("Player has died.");
        }
    }
}
```
