# SyncEvent

Synchronized Event, or most commonly referred as "SyncEvent" are easily definable in your code, and will handle automatically calling events to all [players](../../../terminology/playerid-client-connection.md) listening to the event.

Working with SyncEvent is as easy as using a regular UnityEvent, only having to be mindful of who has authority.

SyncEvents are built with the Network Module setup, meaning that you have to initialize it. Below is a usage example:

```csharp
//Creates an instance of the event - True means that it is owner authority
[SerializeField] private SyncEvent<int> syncEvent = new(true);

protected override void OnSpawned()
{
    //Listening to the event
    syncEvent.AddListener(SyncEventTest);
}

private void SyncEventTest(int myValue)
{
    //Everyone subscribed to the syncEvent will receive this value when the owner invokes it
    Debug.Log($"Received value: {myValue} from SyncEvent");
}

public void InvokeSyncEvent()
{
    //Because the event is owner auth, only the owner can call the event.
    if (!isOwner)
        return;
    
    //Invoking the event as the owner with the value 10
    syncEvent.Invoke(10);
}
```

The SyncEvent is serializable in the Unity editor as a Unity Event, meaning that you can easily add events straight from the inspector view.

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>
