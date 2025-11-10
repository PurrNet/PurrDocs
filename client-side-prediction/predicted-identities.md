# Predicted Identities

Predicted identities are regular Unity components that PurrDiction simulates deterministically. The system snapshots and reconciles their `STATE` against server authority, while your code focuses on clean, single‑player‑style logic.

**Variants**

- `PredictedIdentity<STATE>`
  - For stateful systems without direct player input (AI, timers, pooled objects).
  - Implement: `GetInitialState`, `GetUnityState`, `SetUnityState`, `Simulate`, `LateSimulate` (optional), `UpdateView` (optional).
- `PredictedIdentity<INPUT, STATE>`
  - Adds an `INPUT` pipe for local/remote control. Implement: `GetFinalInput`, `UpdateInput` (optional), `SanitizeInput`, `Simulate(INPUT, ref STATE, float)`, `ModifyExtrapolatedInput` (optional).
- `StatelessPredictedIdentity`
  - For pure event/logic systems that don’t carry custom state. Override `Simulate(float delta)`.

Both `INPUT` and `STATE` must be structs that implement the appropriate prediction interfaces (`IPredictedData`, `IPredictedData<T>`).

***

**Lifecycle Hooks**

- `LateAwake()` — Called once after fresh spawn initialization. View‑only setup is appropriate here.
- `SimulationStart()` — First tick only, before simulation begins. Good for caching or one‑time transitions.
- `Simulate(...)` — Deterministic simulation each tick. Use only `STATE`/`INPUT` and deterministic data.
- `LateSimulate(...)` — Optional late pass after `Simulate` each tick (e.g., composing derived values).
- `Destroyed()` — Called when despawning/cleanup occurs (pool or destroy).
- `ResetState()` — Clears ownership/IDs and resets interpolation and internal caches for pooling.

Unity bridging:

- `GetUnityState(ref STATE state)` — Read Unity components into `STATE`.
- `SetUnityState(STATE state)` — Apply `STATE` back to Unity components after rollback.

View & interpolation:

- `UpdateView(STATE viewState, STATE? verified)` — Render using the current interpolated `viewState`. `verified` holds the last server snapshot, when present.
- `ResetInterpolation()` — Clear internal smoothing/error.

***

**Ownership and Control**

- `owner` — Optional `PlayerID` who owns this identity.
- `isOwner` — True if `owner == PredictionManager.localPlayer` on this client.
- `isController` — True for the owner on clients; on server, also true for bots/AI cases.
- `OnViewOwnerChanged(oldOwner, newOwner)` — View‑only callback when ownership changes; do not mutate simulation here.

Use these flags for visuals (camera, highlights, UI). Keep simulation deterministic and independent of local presentation.

***

**Example Skeleton (Stateful with Input)**

```csharp
public struct MyInput : IPredictedData {
    public float x, y; public void Dispose() {}
}

public struct MyState : IPredictedData<MyState> {
    public Vector3 pos; public Quaternion rot; public void Dispose() {}
}

public class MyPredicted : PredictedIdentity<MyInput, MyState>
{
    protected override MyState GetInitialState() => new MyState {
        pos = transform.position, rot = transform.rotation
    };

    protected override void GetUnityState(ref MyState s)
    { s.pos = transform.position; s.rot = transform.rotation; }

    protected override void SetUnityState(MyState s)
    { transform.SetPositionAndRotation(s.pos, s.rot); }

    protected override void GetFinalInput(ref MyInput i)
    { i.x = Input.GetAxisRaw("Horizontal"); i.y = Input.GetAxisRaw("Vertical"); }

    protected override void SanitizeInput(ref MyInput i)
    { var v = Vector2.ClampMagnitude(new Vector2(i.x, i.y), 1f); i.x = v.x; i.y = v.y; }

    protected override void Simulate(MyInput i, ref MyState s, float dt)
    { s.pos += new Vector3(i.x, 0, i.y) * dt * 5f; }

    protected override void UpdateView(MyState view, MyState? verified)
    { transform.SetPositionAndRotation(view.pos, view.rot); }
}
```
