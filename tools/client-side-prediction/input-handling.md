# Input Handling

Input handling in PurrDiction centers around the generic `PredictedIdentity<INPUT, STATE>` base class. Implement `GetFinalInput(ref INPUT input)` to gather per‑frame input, and optionally `UpdateInput(ref INPUT input)` to cache edge‑triggered inputs (e.g., key down) from Unity’s frame loop.

***

**Key Concepts**

- `INPUT` must be a struct implementing `IPredictedData`.
- Implement `protected virtual void GetFinalInput(ref INPUT input)` to populate inputs deterministically each tick.
- Use `protected virtual void UpdateInput(ref INPUT input)` inside Unity’s frame loop to accumulate one‑shot inputs.
- Use `protected virtual void SanitizeInput(ref INPUT input)` to clamp or normalize values for determinism.

***

**Basic Example**

```csharp
public struct SimpleWASDInput : IPredictedData {
    public NormalizedFloat horizontal;
    public NormalizedFloat vertical;
    public bool jump;
    public bool dash;
    public void Dispose() {}
}

public class SimpleCC : PredictedIdentity<SimpleWASDInput, SimpleCCState>
{
    protected override void GetFinalInput(ref SimpleWASDInput input)
    {
        input.horizontal = Input.GetAxisRaw("Horizontal");
        input.vertical   = Input.GetAxisRaw("Vertical");
        input.dash       = Input.GetKey(KeyCode.LeftShift);
    }

    protected override void UpdateInput(ref SimpleWASDInput input)
    {
        // Edge-triggered input is cached here and consumed once per tick
        input.jump |= Input.GetKeyDown(KeyCode.Space);
    }

    protected override void SanitizeInput(ref SimpleWASDInput input)
    {
        var move = Vector2.ClampMagnitude(new Vector2(input.horizontal, input.vertical), 1f);
        input.horizontal = move.x;
        input.vertical   = move.y;
    }
}
```

***

**Extrapolation and Repeat**

- Remote players can use extrapolated input if the latest input is missing.
- Control behavior via fields on `PredictedIdentity<INPUT, STATE>`:
  - Extrapolate Input: enables extrapolation for remote input.
  - Repeat Input Factor: caps how many ticks a prior input can be reused.
- Override `ModifyExtrapolatedInput(ref INPUT input)` to disable non‑continuous inputs during extrapolation (e.g., `jump = false`).

***

**Why This Pattern**

- Determinism: All input used in simulation is captured, sanitized, and stored per tick.
- One‑shot safety: Edge‑triggered inputs are gathered in `UpdateInput` and consumed once, preventing repeats.
- Flexibility: Works with any control scheme while keeping authoritative reconciliation stable.
