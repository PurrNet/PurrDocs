# SyncHashset

Synchronized HashSet, or most commonly referred as "SyncHashSet" are easily definable in your code, and will handle automatically align a list between all [players](../../playerid-client-connection.md).

Working with SyncHashSet is as easy as using a regular HashSet, only having to be mindful of who has authority.

SyncHashSet are built with the Network Module setup, meaning that you have to initialize it. Below is a usage example:

```csharp
//Creates a new instance of the Hashset - `true` means it is owner auth. 
[SerializeField] private SyncHashSet<string> myHashSet = new(true);

protected override void OnSpawned()
{
    //Subscribing to changes made to the hash set
    myHashSet.onChanged += OnHashSetChanged;
}

private void OnHashSetChanged(SyncHashSetChange<string> change)
{
    //This is called for everyone when the hash set changes.
    //It will log out the Value and operation
    Debug.Log($"HashSet updated: {change}");
}
```

The HashSet is serialized in the Unity editor as a list, for easy debugging. You can't modify values here.

<figure><img src="../../../.gitbook/assets/Unity_SyncHashSet (1).png" alt=""><figcaption></figcaption></figure>
