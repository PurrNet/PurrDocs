# Deterministic Identity

`DeterministicIdentity<STATE>` is a variant designed for strict, bit‑stable simulation. It mirrors the lifecycle of `PredictedIdentity<STATE>` but uses deterministic math (`sfloat`) and can validate state equality across client/server when enabled.

***

**When to Use**

* Systems that can produce identical results on all machines from the same inputs (e.g., strategy sim, deterministic AI, fixed math gameplay).

If you don’t require strict bit‑determinism, prefer [`PredictedIdentity<STATE>`](predicted-identities.md) for simpler float‑based logic.

***

**Key Properties**

* Uses `sfloat` delta in `Simulate`/`LateSimulate` for deterministic time steps.
* History, rollback, and interpolation are the same pattern as stateful identities.
* Networking: by default it doesn’t send state each tick; instead, you can turn on validation to compare states.

Prediction Manager setting:

* Validate Deterministic Data (bool): when enabled, the server writes each deterministic state and clients read+compare it. Mismatches log an error and trigger a `Debug.Break()` to help diagnose.

***

**Overrides**

* `protected virtual void GetUnityState(ref STATE state)` / `protected virtual void SetUnityState(STATE state)`
* `protected virtual void SimulationStart()`
* `protected virtual void Simulate(ref STATE state, sfloat delta)`
* `protected virtual void LateSimulate(ref STATE state, sfloat delta)`
* `protected virtual void UpdateView(STATE viewState, STATE? verified)`
* `protected virtual STATE Interpolate(STATE from, STATE to, float t)`

Note the `sfloat` delta — use deterministic math throughout your simulation. You can also mix in `FP` for fixed point math.

***

**STATE Requirements**

* `STATE : struct, IPredictedData<STATE>` — same as other identities.
* Provide deterministic operations via `IMath<STATE>` (used by default interpolation): `Add`, `Scale`, `Negate`.
* Avoid non‑deterministic data (raw `float` computation); prefer `sfloat` or integer/fixed‑point math inside state operations.

***

**Example**

```csharp
using PurrNet.Prediction;

public struct OscState : IPredictedData<OscState> {
    public sfloat phase; public sfloat speed; public sfloat amplitude;
    public void Dispose() {}
}

public class Oscillator : DeterministicIdentity<OscState>
{
    protected override OscState GetInitialState() => new OscState {
        speed = (sfloat)1.5f, amplitude = (sfloat)2f, phase = 0
    };

    protected override void GetUnityState(ref OscState s) { /* read if needed */ }
    protected override void SetUnityState(OscState s) { /* apply on rollback if needed */ }

    protected override void Simulate(ref OscState s, sfloat dt)
    {
        s.phase += s.speed * dt;
        var y = sfloat.Sin(s.phase) * s.amplitude;
        // use y for downstream deterministic effects; drive visuals in UpdateView
    }

    protected override void UpdateView(OscState view, OscState? verified)
    {
        // Convert to float for rendering only
    }
}
```

***

**Best Practices**

* Use `sfloat` and integer math for all simulation‑impacting calculations.
* Avoid sampling Unity time or random APIs directly; use `PredictedTime` and `PredictedRandom`.
* Keep any conversions to `float` purely in `UpdateView`.
* Enable Validate Deterministic Data only during development; it adds extra packing/comparison overhead.
