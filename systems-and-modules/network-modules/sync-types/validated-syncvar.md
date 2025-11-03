# Validated SyncVar

This is a more advanced variation of the [SyncVar](syncvar.md) which allows you to get the best of both world in terms of client sided responsiveness with server authorized changes.

{% hint style="warning" %}
This module is automatically owner-authenticated and bound to the **NetworkIdentity** it resides in.
{% endhint %}

The idea of the validated SyncVar is quite easy:\
\- Client makes local change and gets immediate response\
\- Server is notified of this change and can validate whether the change should be validated\
\- If the change is validated, it's sent to everyone\
\- If the change is not validated, it's returned to the owner gets a callback

This allows you to easily add cheat proofing to the Validated SyncVar whilst still keeping it fully responsive in fair/valid scenarios.

Below is a simple example usage, which only allows the value to be counted up, but never down.&#x20;

```csharp
private ValidatedSyncVar<int> _testVar = new();

private void OnEnable()
{
    _testVar.serverValidation += ServerValidator;
    _testVar.onValidationFail += OnValidationFail;

    _testVar.onChangedWithOld += OnChanged;
}

private void OnDisable()
{
    _testVar.serverValidation -= ServerValidator;
    _testVar.onValidationFail -= OnValidationFail;
    
    _testVar.onChangedWithOld -= OnChanged;
}

private bool ServerValidator(int oldValue, int newValue)
{
    //Only the server will reach this. This is called whenever a change occurs. 
    if (oldValue > newValue)
        return false;
    
    return true;
}

private void OnValidationFail(int failedValue, int authoritativeValue)
{
    //Only the owner will receive this if the validation fails
    Debug.Log($"Validation failed on {failedValue}. Returning to {authoritativeValue}");
}

private void OnChanged(int oldValue, int newValue, bool validated)
{
    //This is called immediately with validated false for the owner, 
    //and everyone received with validated = true if it goes through
    Debug.Log($"Value changed: {oldValue} -> {newValue} | Validated: {validated}");
}
```

### Behavior & Expectations

* Authority
  * Uses `parent.IsController(true)` for writes.
  * If `owner` is null, only server can write.
  * Module is owner-authed and bound to its `NetworkIdentity`.
* Events
  * `onChanged(old, new, validated)`
  * `onChangedWithOld(old, new, validated)`
  * Owner emits twice on success: first with `validated=false` (local predict), then `validated=true` (server accept).
  * On reject, owner emits `validated=true` when reverting to authoritative, and fires `onValidationFail(failed, authoritative)`.
  * Non-owners receive only one event with `validated=true`.
  * [Host](../../../terminology/host.md) also emits predict then validated (consistent with clients).
* Validation
  * `serverValidation` is multicast; all handlers must return true to accept.
  * Runs on server for every candidate change (including host).
* IDs & Ordering
  * Each candidate has a `PackedUInt` packetId.
  * Client ignores `AcceptOwner/RejectOwner` with `packetId < _pendingId` (stale).
  * This keeps the display at the most recent local prediction until the matching latest ack arrives.
* Common Scenarios
  * Owner set accepted: owner gets `(old→new, false)` then `(old→new, true)`; others get `(old→new, true)`.
  * Owner set rejected: owner gets `(old→candidate, false)` then `(candidate→authoritative, true)` + `onValidationFail(candidate, authoritative)`; others get nothing.
  * Rapid 1→2→3 with latency: owner shows 3 immediately; stale acks for 1/2 are ignored; only ack for 3 applies.
  * Server sets value (no prediction): everyone gets a single `(old→new, true)`. Owner does not get a `false` event in this path.
  * No owner: only server may set; client attempts are rejected.
  * Owner change: non-owner listeners subscribe to authoritative stream; pending owner state is cleared on change.
  * Late joiner: receives current authoritative via inner `SyncVar`; only a single `(old→new, true)` if different.
