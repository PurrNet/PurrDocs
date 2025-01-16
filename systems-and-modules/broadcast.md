# Broadcast

Broadcasting in networking systems (like PurrNet) allows you to easily send and receive data without requiring network identities. It can be useful in certain systems and situations, however, for most setups, it will be just as good and easier to work with [RPC's](remote-procedure-call-rpc/).

{% embed url="https://youtu.be/bwIiKxbK-HE" %}

In order to work with RPC's, you need 3 things:

* Custom data structure to send
* Subscribing to the broadcasting event
* Sending the data

The client can only broadcast directly to the server, where as the server/host can broadcast to everyone.

A basic broadcasting setup will look something like this:

```csharp
private void Start()
{
    //We subscribe to receiving this type of data
    InstanceHandler.NetworkManager.Subscribe<DataToSend>(OnReceiveData);
    //Below is an example of how you can subscribe only as the server. False for client
    //This can potentially give you some more control
    //InstanceHandler.NetworkManager.Subscribe<DataToSend>(OnReceiveData, true);
}

private void OnDestroy()
{
    //Remember to unsubscribe again to avoid possible errors and issues
    InstanceHandler.NetworkManager.Unsubscribe<DataToSend>(OnReceiveData);
}

private void Update()
{
    //Press the 1 key to send data
    if (Input.GetKeyDown(KeyCode.Alpha1))
    {
        if (InstanceHandler.NetworkManager.isServer)
        {
            //If we are the server, we build the data and send it to every player
            var data = new DataToSend()
            {
                intValue = 69,
                boolValue = false
            };
            InstanceHandler.NetworkManager.SendToAll(data);
        }
        else
        {
            //If we are not the server, we send some data to the server
            var data = new DataToSend()
            {
                intValue = 420,
                boolValue = true
            };
            InstanceHandler.NetworkManager.SendToServer(data);
        }
        
    }
}

private void OnReceiveData(PlayerID player, DataToSend data, bool asserver)
{
    //We've now received data
    Debug.Log($"Received data from player {player}: {data.intValue}, {data.boolValue}");
}

//We need to know what data type we are sending around
private struct DataToSend : IPackedAuto
{
    public int intValue;
    public bool boolValue;
}
```
