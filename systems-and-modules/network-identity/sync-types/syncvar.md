# SyncVar

Synchronized variables, or most commonly referred as "SyncVar" are easily definable in your code, and will handle automatically align a script variable between all [players](../../playerid-client-connection.md).&#x20;

{% embed url="https://youtu.be/35N_nfny6Ec" %}

SyncVars are built with the Network Module setup, meaning that you have to initialize it. Below is a usage example:

```csharp
private SyncVar<int> mySync = new(2); //Now defaults to 2
private SyncVar<int> myOtherSync = new(5, ownerAuth:true); //Defaults to 5, and is owner auth

protected override void OnSpawned(bool asServer)
{
    mySync.onChanged += OnMySyncChange;

    if (asServer)
        mySync.value = 420;

    if (isOwner)
        myOtherSync.value = 69;
}

private void OnMySyncChange(int newValue)
{
    Debug.Log("SyncVar has changed to: " + newValue);
}
```

When populating the SyncVar, you can do a few things:

* Feed it nothing: `new();`
* Feed it a default: `new(5);` //In case of number
* Feed it settings like `ownerAuth` or `sendIntervalInSeconds`

The owner auth setting default value can be found in your active network rules.
