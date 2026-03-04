---
icon: clock
---

# Tick System

The Tick System in PurrNet provides a synchronized clock between the server and all clients. Instead of relying on frame rate (which varies per machine), the tick system gives you a consistent time reference that the entire network agrees on. This is useful for synchronizing gameplay logic, timing abilities, and anything else that needs to be frame-rate independent across machines.

You can access the tick module through the Network Manager:

```csharp
var tickModule = InstanceHandler.NetworkManager.tickModule;
```

## Tick Events

The tick module fires events every network tick, giving you a fixed-rate update loop similar to `FixedUpdate` but synced across the network:

```csharp
private void Awake()
{
    var nm = InstanceHandler.NetworkManager;
    nm.onTick += OnTick;
    nm.onPreTick += OnPreTick;
    nm.onPostTick += OnPostTick;
}

private void OnDestroy()
{
    var nm = InstanceHandler.NetworkManager;
    nm.onTick -= OnTick;
    nm.onPreTick -= OnPreTick;
    nm.onPostTick -= OnPostTick;
}

private void OnPreTick()
{
    // Runs before tick processing
}

private void OnTick()
{
    // Main tick logic
}

private void OnPostTick()
{
    // Runs after tick processing
}
```

{% hint style="info" %}
On a host, tick events fire twice per tick: once for the server and once for the client.
{% endhint %}

## Useful Properties

| Property | Type | Description |
| --- | --- | --- |
| `localTick` | `int` | The client's tick count since connecting |
| `syncedTick` | `int` | Server-aligned tick, synchronized across all clients |
| `rtt` | `float` | Round-trip time (ping) in seconds |
| `tickRate` | `int` | How many ticks run per second |
| `floatingPoint` | `float` | Fractional tick value for precise timing within the current update |
| `rollbackTick` | `double` | Synced tick adjusted for half the RTT. Used for [Collider Rollback](collider-rollback.md) |

## Time Conversion

The tick module includes helper methods for converting between ticks and seconds:

```csharp
var tickModule = InstanceHandler.NetworkManager.tickModule;

// Convert ticks to seconds
float seconds = tickModule.TickToTime(100);

// Convert seconds to ticks
int ticks = tickModule.TimeToTick(5.0f);

// For more precision using fractional ticks
float preciseSeconds = tickModule.PreciseTickToTime(100.5);
double preciseTicks = tickModule.TimeToPreciseTick(5.0f);
```

## Common Use Cases

**Cooldown tracking:** Use `syncedTick` to track ability cooldowns that stay in sync regardless of latency.

```csharp
private int _lastAttackTick;
private int _cooldownTicks = 30; // at 30 tick rate, this is 1 second

private void TryAttack()
{
    var currentTick = InstanceHandler.NetworkManager.tickModule.syncedTick;

    if (currentTick - _lastAttackTick < _cooldownTicks)
        return;

    _lastAttackTick = currentTick;
    PerformAttack();
}
```

**Lag compensation:** The `rollbackTick` property is what you pass to [Collider Rollback](collider-rollback.md) to validate hits accounting for the player's ping.
