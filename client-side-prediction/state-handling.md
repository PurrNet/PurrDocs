# State Handling

`STATE` is the single source of truth for your simulation. PurrDiction records, reconciles, and interpolates states so clients stay responsive while remaining consistent with server authority.

***

**Requirements**

- `STATE` must be a struct implementing `IPredictedData<STATE>` (which extends `IMath<STATE>` and `IPackedAuto`).
- Provide deterministic math on your state via `IMath<T>` methods (the default linear interpolation uses `Add`, `Scale`, `Negate`).

***

**What Goes in STATE**

- Simulation‑critical data only: positions, rotations, velocities, timers, flags; anything that affects future ticks.
- Do not store transient view data; compute those in `UpdateView`.

***

**Lifecycle and Reconciliation**

- Simulation loop writes `currentState` each tick.
- PurrDiction saves snapshots and reconciles against server frames as they arrive.
- On rollback:
  - `ReadState` loads the verified snapshot.
  - `Rollback(tick)` applies it and calls `SetUnityState(state)` for external components.
  - `UpdateRollbackInterpolationState(delta, accumulateError)` adjusts view smoothing.

Unity bridging:

- `protected override void GetUnityState(ref STATE state)` — Read Unity → STATE when needed.
- `protected override void SetUnityState(STATE state)` — Write STATE → Unity on rollback.

***

**View vs Verified**

- `viewState` is the smoothed state to render this frame.
- `verifiedState` exposes the most recent authoritative snapshot, when present.
- Override `UpdateView(STATE viewState, STATE? verified)` to drive visuals.
- Call `ResetInterpolation()` to clear accumulated error (e.g., teleports).

***

**Example: Minimal STATE**

```csharp
public struct MyState : IPredictedData<MyState>
{
    public Vector3 pos; public Quaternion rot;
    public void Dispose() {}
    // IMath<MyState> default ops come from packer codegen; override Interpolate if non-linear
}

public class Mover : PredictedIdentity<MyState>
{
    protected override MyState GetInitialState() => new MyState { pos = transform.position, rot = transform.rotation };
    protected override void GetUnityState(ref MyState s) { s.pos = transform.position; s.rot = transform.rotation; }
    protected override void SetUnityState(MyState s) { transform.SetPositionAndRotation(s.pos, s.rot); }
    protected override void Simulate(ref MyState s, float dt) { /* mutate s deterministically */ }
    protected override void UpdateView(MyState view, MyState? verified) { transform.SetPositionAndRotation(view.pos, view.rot); }
}
```
