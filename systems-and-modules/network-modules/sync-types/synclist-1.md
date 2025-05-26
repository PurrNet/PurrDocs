# SyncArray

A Synchronized Array, more commonly referred to as a "SyncArray", is easily definable in your code. It will handle automatically aligning the contents of an array between all [players](../../../terminology/playerid-client-connection.md).

Working with SyncArrays is as easy as using a regular array, you have only to be mindful of who has authority over it.

SyncArrays are built with the Network Module setup, meaning that you have to initialize it. Below is a usage example:

```csharp
//Creates a new instance of the array
//20 sets the initial length of the array
//`true` means it is owner auth. 
public SyncArray<int> syncArray = new(20, true);

protected override void OnSpawned()
{
    //Subscribing to changes made to the array
    syncArray.onChanged += OnArrayChange;
}

private void OnArrayChange(SyncArrayChange<int> change)
{
    //This is now called for everyone when the array changes.
    //It will log out the value, index and operation
    Debug.Log(change);
}

private void ChangeMyArray()
{
    //This will change or add a value
    syncArray [0] = 69;
    
    //Resized the array to 15 elements
    syncArray.Length = 15;
}
```
