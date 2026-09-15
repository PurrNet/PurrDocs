---
version: ">=1.4.0"
---

# Lag Compensation

A player shoots at what they see on their screen, and what they see is a little behind the simulation: view interpolation buffers a few ticks so movement looks smooth. By the time the input reaches the server, the target has moved on. Lag compensation rewinds hit tests to the tick the shooter was actually looking at, so shots land where the shooter saw the target.

PurrDiction builds this on top of PurrNet's [Collider Rollback](../systems-and-modules/collider-rollback.md) and adds the part PurrNet cannot know on its own: which prediction tick each player was presenting when they produced an input.

## Setup

1. Add a `ColliderRollback` component to every object that should be hittable. PurrNet records its collider history from there; predicted targets need nothing else.
2. Nothing is required on the shooter.

The Prediction Manager has a **Lag Compensation** section with two settings:

* **Max Lag Compensation Seconds** caps how far back a client may ask the server to rewind. Requests beyond it are clamped. The view interpolation buffer never holds more than 0.1 s, so the default of 0.15 s covers legitimate clients with headroom. Lower it if you want to limit how much a laggy shooter can hit a target that already moved to cover.
* **Forward View Offsets** sends every player's view offset to the other clients inside verified frames, so they replay other players' shots with the exact rewind the server used. It costs about 10 bits per player per tick. Turned off, clients estimate the rewind for other players' shots instead; the server's result is the same either way.

## Firing from Simulate

Inside `Simulate`, the identity exposes `lagCompensationTick`: the precise prediction tick the controlling player was presenting when they produced the input for the tick being simulated. Pass it to the queries on `predictionManager.lagCompensation`:

```csharp
public class HitscanShooter : PredictedIdentity<FireInput, HitscanState>
{
    [SerializeField] private float _range = 100f;
    [SerializeField] private LayerMask _hitMask = ~0;

    protected override void Simulate(FireInput input, ref HitscanState state, float delta)
    {
        if (!input.fire)
            return;

        var ray = new Ray(transform.position, transform.forward);

        if (predictionManager.lagCompensation.Raycast(lagCompensationTick, ray, out var hit, _range, _hitMask))
        {
            state.hits += 1;
            state.lastHitPoint = hit.point;
        }
    }
}
```

That is the whole pattern. The same `Simulate` code runs on every peer:

* On the server the query is authoritative. The rewind comes from the offset the client reported with its input, clamped to the manager's maximum.
* On the shooter's own client the query runs against that client's own recorded history, which is exactly what it rendered, so the hit is predicted locally with no round trip.
* On other clients the query uses the forwarded offset when **Forward View Offsets** is on, or an estimate otherwise. Either way the verified frame corrects the result if it differs.

Server-controlled identities, such as bots, have no view offset; for them `lagCompensationTick` is simply the tick being simulated. The property is only meaningful inside `Simulate`.

Because the query is part of the simulation, the outcome lives in predicted state and rolls back and replays like anything else. You do not need a `ServerRpc`, a rollback tick in your input, or any client-to-server validation call; sending the fire button is enough.

## How this differs from PurrNet's rollbackTick

`predictionManager.rollbackTick` is PurrNet's `NetworkIdentity.rollbackTick`, which every identity has: the synced precise tick minus half the measured round trip, in the tick manager's own tick space. It is a latency estimate meant for the classic flow where a client raycasts locally, sends the tick in a `ServerRpc`, and the server rewinds by it.

`lagCompensationTick` is a different quantity:

* It is in prediction tick space. Client prediction ticks are already stamped with the server ticks they execute at, so round trip is not what needs compensating. What needs compensating is how far behind the live tick the player's screen was, which is the view interpolation offset.
* It is per identity, not per peer. It answers "what was this identity's controller looking at when they produced this tick's input", and the server derives it from the offset that client uploaded with the input, clamped to **Max Lag Compensation Seconds**.
* It is recorded per tick, so it replays and rolls back deterministically. It is only meaningful inside `Simulate`.

Use `lagCompensationTick` with `predictionManager.lagCompensation` for predicted identities. `rollbackTick` remains the right input for direct `RollbackModule` calls from ordinary networked code.

## Available queries

`predictionManager.lagCompensation` mirrors the `RollbackModule` query surface, addressed in prediction ticks. Every method takes the tick first and then the usual Unity arguments:

| 3D | 2D |
| --- | --- |
| `Raycast`, `SphereCast`, `BoxCast`, `CapsuleCast` (single hit and array overloads) | `Raycast`, `CircleCast`, `BoxCast`, `CapsuleCast` (single hit and array overloads) |
| `SphereOverlap`, `BoxOverlap`, `CapsuleOverlap` | `CircleOverlap`, `BoxOverlap`, `CapsuleOverlap` |
| `CheckSphere`, `CheckBox`, `CheckCapsule` | `CheckCircle`, `CheckBox`, `CheckCapsule` |

The queries do not physically move colliders. As with PurrNet's rollback, the ray is bent per collider against its recorded history, so no physics events fire and nothing in the live scene is disturbed.

If you need a `RollbackModule` method that has no wrapper, resolve the tick yourself:

```csharp
if (predictionManager.lagCompensation.TryResolve(lagCompensationTick, out var module, out var rollbackTick))
{
    // rollbackTick is the precise tick the RollbackModule records against
}
```

Prediction ticks and PurrNet's tick counter drift apart on lead adjustments and hitches, so the manager records the mapping every simulated tick rather than assuming a constant offset. `TryResolve` returns false when collider rollback is unavailable for the scene or no tick has been recorded yet. A tick outside the recorded window is extrapolated from the nearest entry, and whether the `RollbackModule` still has collider history for it is then up to PurrNet's own rollback window.

## Choosing when to use it

Lag compensation makes sense when the server owns the target's movement, which is the case for predicted identities: clients propose input and the server simulates the result. The shooter gets favored, since they hit what they saw; the target may get hit a moment after they thought they reached cover. **Max Lag Compensation Seconds** is your knob for how far that favor goes.

For melee, overlap checks, or anything where the attacker and target are both close and predicted, the ordinary live physics query inside `Simulate` is often good enough. Reach for lag compensation when the projectile is instant and the distance makes the view delay visible.

## Related diagnostics

The manager exposes a few read-only values that help when debugging rewinds:

* `viewTick` is the precise prediction tick this client is presenting, including the fraction between two ticks. It lags `localTick` by the interpolation buffer depth.
* `verifiedServerTick` is the newest server tick whose authoritative frame this client has applied.
* `lead` is how many ticks the predicted head runs ahead of the verified tick.
* `maxLagCompensationTicks` is the configured cap converted to ticks.
* `GetLagCompensationTick(player, tick)` is what `lagCompensationTick` calls under the hood, if you need it for another player or tick.
