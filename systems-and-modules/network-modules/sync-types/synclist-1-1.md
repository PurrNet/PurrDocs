# SyncQueue

A Synchronized Queue, more commonly referred to as a "SyncQueue", is easily definable in your code. It will handle automatically aligning the contents of a list between all [players](../../../terminology/playerid-client-connection.md).

Working with a SyncQueue is as easy as using a regular queue, you have only to be mindful of who has authority over it.

SyncQueue's are built with the Network Module setup, meaning that you have to initialize it. Below is a usage example:

```csharp
//Creates a new instance of the queue - `true` means it is owner auth. 
[SerializeField] private SyncQueue<int> myQueue = new(true);

protected override void OnSpawned()
{
    //Subscribing to changes made to the list
    myQueue .onChanged += OnQueueChanged;
}

private void OnQueueChanged(SyncQueueChange<int> change)
{
    //This is called for everyone when the queue changes.
    //It will log out the value and operation
    Debug.Log($"Queue updated: {change}");
}

private void ChangeMyQueue()
{
    //This will enqueue into the queue
    myQueue.Enqueue(69);
    
    //This will dequeue the first element in the queue
    myQueue.Dequeue();
    
    //This will clear the queue
    myQueue.Clear();
    
    //This will peek at the first element
    var myVal = myQueue.Peek();
}
```

The SyncQueue is serialized in editor. It will also attempt to auto serialize non-serializable classes/structs by grabbing the ToString().

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

### SyncQueueChange

When working with the SyncQueueChange which comes with the onChanged callback, you have some basic information available to you.

The `.operation` tells you what the change is, that can be: Enqueued, Dequeued, Cleared.

The  `.value` is the new value which it has been changed. This is of type T to match the type of the SyncQueue.
