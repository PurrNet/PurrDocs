# Security Model

Client-side prediction improves responsiveness; it does not grant authority. The server receives player input, sanitizes it, runs the authoritative simulation, and sends verified state back to observers.

## What the client controls

An owning client can propose input for its predicted identities. It cannot directly declare an authoritative position, health value, spawn, hit, or physics result through the prediction state pipeline.

The server accepts input in the context of ownership and simulates the outcome itself. A modified client can still send malicious but correctly shaped input, so serialization and range checks are only the first layer of validation.

## Sanitize input

Override `SanitizeInput` to normalize or reject values before server simulation:

```csharp
protected override void SanitizeInput(ref PlayerInput input)
{
    input.move = Vector2.ClampMagnitude(input.move, 1f);

    if (!float.IsFinite(input.aimRadians))
        input.aimRadians = 0f;
}
```

Then validate intent against authoritative state inside simulation:

* Enforce cooldowns, ammo, stamina, and movement limits.
* Confirm the identity is in a state where the action is legal.
* Derive hits and collisions from server simulation rather than client claims.
* Rate-limit expensive actions and predicted hierarchy creation.

## Missing and repeated input

The server can extrapolate missing input depending on the Prediction Manager's Input Queue Settings. Strip edge-triggered actions in `ModifyExtrapolatedInput` so a missing frame cannot repeat jump, fire, interact, or purchase commands.

Input queue limits are networking safeguards, not gameplay validation. Your simulation still needs domain-specific limits.

## Policies do not change authority

[Prediction policies](prediction-policies.md) change only how a client presents and reconciles an identity:

* `ServerRelay` does not make a client authoritative; it reduces speculative client simulation.
* `SoftCorrection` does not trust the client's result; verified server state remains the correction target.
* `PredictedIfOwned` predicts the local owner's copy for responsiveness, while the server still verifies it.

Do not use a smoother correction policy as evidence that a state is secure or valid.

## Side effects and ordinary networking

Prediction simulation can replay. Sending RPCs, granting rewards, writing persistence, or invoking external systems directly from an unguarded simulation callback can execute more than once.

Keep authoritative rewards and persistence on the server, make them idempotent where possible, and trigger presentation with `PredictedEvent` or `predictionManager.isVerifiedView`.

`PredictedIdentitySpawner` is intentionally marked `[PredictionUnsafe]` because it bridges predicted state to ordinary PurrNet hierarchy side effects. Use it narrowly and let PurrNet's network rules, visibility, and ownership checks govern the spawned identities.
