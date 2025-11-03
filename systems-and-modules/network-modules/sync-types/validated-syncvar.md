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
