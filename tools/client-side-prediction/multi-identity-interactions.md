---
icon: link
---

# Interacting With Multiple Identities

Many features require one predicted identity to read or affect another (e.g., projectiles applying knockback, an ability system modifying player state, or a shared input bus). This guide covers safe, deterministic patterns to discover, read, and modify other identities and their input.

***

**Discovery and References**

- Prefer stable handles over direct `GameObject` references when objects can be created/destroyed under prediction.
- Use the Predicted Hierarchy to resolve components from IDs:

  - `TryCreate(...)` / `Create(...)` → returns `PredictedObjectID`
  - `GetComponent<T>(PredictedObjectID?)` → resolve an identity or component at runtime
  - `TryGetId(GameObject, out PredictedObjectID)` → convert a known object into a stable ID

- For long‑lived relationships, store `PredictedObjectID` in your `STATE` so it survives rollback/replay.

```csharp
public struct AbilityState : IPredictedData<AbilityState>
{
    public PredictedObjectID target;
    public void Dispose() {}
}

// Later during simulate
var targetCtrl = predictionManager.hierarchy.GetComponent<MyController>(state.target);
```

***

**Reading and Writing Another Identity’s STATE**

- You may read or modify another identity’s state during simulation. Keep it deterministic and do it only while simulating.
- Access the other identity as its concrete type and mutate via its `currentState`:

```csharp
// inside Simulate(...)
var target = predictionManager.hierarchy.GetComponent<MyMover>(state.target);
if (target != null)
{
    ref var ts = ref target.currentState; // by ref
    ts.velocity += knockback;             // deterministic mutation
}
```

Tips:

- Only mutate in `Simulate`/`LateSimulate` (the simulation phase). You can annotate helpers with `[SimulationOnly]` to ensure they only run while simulating.
- Keep inter‑identity logic order‑independent when possible. If order matters, consolidate logic into a single orchestrator identity.

***

**Using Another Identity’s Input**

- You can read another identity’s input for the current tick via its `currentInput` (for identities with input).
- Use this sparingly; prefer sharing intent/state rather than raw input when possible.

```csharp
var player = predictionManager.hierarchy.GetComponent<PlayerController>(state.player);
if (player != null)
{
    var input = player.currentInput; // read‑only view for this tick
    // derive effects from input (e.g., combo triggers, assist behaviors)
}
```

Note: Do not attempt to modify another identity’s input history. Instead, mutate its state or publish events that it reacts to.

***

**Cross‑Identity Events**

- Use `PredictedEvent`/`PredictedEvent<T>` for view‑safe eventing across identities. Events only invoke when it’s valid to do so (server, owner while not replaying, or verified client), preventing double‑fire during replays.

```csharp
// In an identity
private PredictedEvent onHit;

protected override void LateAwake()
{ onHit = new PredictedEvent(predictionManager, this); }

void SimulateHit()
{ onHit.Invoke(); } // will invoke only in valid contexts
```

***

**Ownership and Control**

- If you only want the controller/owner to make a change, gate logic with `isController` or `IsOwner()`.
- For player‑centric lookups and events, use `predictionManager.players` to iterate or react to players joining/leaving.

***

**Create/Destroy + IDs**

- When spawning transient objects (e.g., projectiles), capture their `PredictedObjectID` in the originating identity’s `STATE` if you’ll need to interact with them later.
- Use the Hierarchy to `Delete(id)` deterministically when cleaning up.

***

**Ordering and Determinism**

- The manager updates identities in a consistent order (by object ID then component ID). Avoid relying on a specific order for correctness.
- If strict ordering is required (e.g., system A before B), move the shared mutation into a single identity (an aggregator) and have others read the outcome.

***

**Example: Projectile Applies Knockback**

```csharp
public struct ProjectileState : IPredictedData<ProjectileState>
{
    public PredictedObjectID target; public Vector3 force;
    public void Dispose() {}
}

public class Projectile : PredictedIdentity<ProjectileState>
{
    protected override void Simulate(ref ProjectileState s, float dt)
    {
        if (!s.target.HasValue) return;
        var mover = predictionManager.hierarchy.GetComponent<MyMover>(s.target);
        if (mover == null) return;
        ref var ms = ref mover.currentState;
        ms.velocity += s.force; // deterministic state mutation
        predictionManager.hierarchy.Delete(id.objectId); // self destroy after applying
    }
}
```

