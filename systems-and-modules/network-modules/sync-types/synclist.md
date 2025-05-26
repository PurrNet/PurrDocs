# SyncList

A Synchronized List, more commonly referred to as a "SyncList", is easily definable in your code. It will handle automatically aligning the contents of a list between all [players](../../../terminology/playerid-client-connection.md).

Working with SyncLists is as easy as using a regular list, you have only to be mindful of who has authority over it.

SyncLists are built with the Network Module setup, meaning that you have to initialize it. Below is a usage example:

```csharp
//Creates a new instance of the list - `true` means it is owner auth. 
[SerializeField] private SyncList<int> myList = new(true);

protected override void OnSpawned()
{
    //Subscribing to changes made to the list
    myList.onChanged += OnListChanged;
}

private void OnListChanged(SyncListChange<int> change)
{
    //This is called for everyone when the list changes.
    //It will log out the value, index and operation
    Debug.Log($"List updated: {change}");
}

private void ChangeMyList()
{
    //This will change or add a value
    myList[0] = 69;
    
    //This will remove the value
    myList.Remove(69);
    
    //This will remove the entry at the given index
    myList.RemoveAt(0);
    
    //This will clear the list
    myList.Clear();
    
    //This will insert a value at the given index
    myList.Insert(1, 420f);
    
    //This will mark the index as dirty
    myList.SetDirty(0);
}
```

The SyncList is serialized in editor, however, you shouldn't edit it here. This is purely for visual debugging

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>
