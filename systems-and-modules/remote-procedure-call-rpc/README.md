# Remote Procedure Call (RPC)

In short, an RPC allows you to call a method on another device/machine. If you're on a client and successfully call a ServerRpc, that method will now be called on the server machine and not on the local machine actually calling the method. This makes it very easy to interact with other machines over the network.

{% embed url="https://youtu.be/h2YRwb0RhcA" %}

### Calling RPC's is easy

In order to call RPC's, you have to make sure your script inherits from _at least_ [Network Identity](../network-identity/), but it is recommended that you inherit from [Network Behaviour](../network-identity/networkbehaviour.md) to get more functionality.

The RPC logic _**can**_ depend on ownership, and this is modifiable in your [Network Rules](../network-manager/network-rules.md).

#### Example script called NetworkTest:

```csharp
using PurrNet;

public class NetworkTest : NetworkBehaviour
{
    //On spawned will be called when the object is first seen by the network.
    //If you are a host, this OnSpawned will be called twice, once for server and once for client
    protected override void OnSpawned(bool asServer)
    {
        if (asServer)
            return;
        
        //If your network rules are set to everyone on ServerRpc, this can be called from all clients
        //Alternatively, it can only be called by the owner of the identity
        TestServerMethod();
    }

    [ServerRpc]
    private void TestServerMethod()
    {
        //This code will be called on the server.
        
        //The server is now calling the observers rpc method on all observing clients.
        //If your Network Rules allow for it, this could also be called from clients, essentially skipping the Server RPC
        TestObserverMethod();
    }

    [ObserversRpc]
    private void TestObserverMethod()
    {
        //This code will be called on every observing clients machine.
        //This is a good way to update all observing clients with the same information.
    }
    
    [TargetRpc]
    private void TestTargetMethod(PlayerID target)
    {
        //This code will only be called on the local instance of the target
        //This is a good way to update a target client
    }
}
```

### Types of RPC's

There are 3 different kinds of RPC's:\
\- ServerRPC\
\- ObserversRPC\
\- TargetRPC

Different RPC's has different parameters to take in, like so:

```csharp
[ServerRpc(RequireOwnership:false, RunLocally:true)]
```

Below you'll find the different RPC types with the parameters they each hold.

#### ServerRPC:

The ServerRPC will call **Client -> Server** meaning that the method which you execute on the client, will be ran on the Server instead.&#x20;

**It holds the following parameters:**

* _**RequireOwnership**_ - This will override any settings within the [Network Rules](../network-manager/network-rules.md) or the [Network Identity](../network-identity/) inspector. With this, you can modify whether ownership is required to call the RPC.
* _**RunLocally**_ - By default, this is **false,** however, if you override it to **true** it will mean that the called will run the logic as well as the server.

#### ObserversRPC:

The ObserversRPC will call **Server -> All clients,** or **Client -> All clients** if your [network rules](../network-manager/network-rules.md) allow it. This means that the method being called will trigger for every client.

**It holds the following parameters:**

* _**RequireServer**_ - This will override any settings within the [Network Rules](../network-manager/network-rules.md) as to whether clients can call the ObserversRpc directly.
* _**BufferLast**_ - If set to true, when a new [client](../playerid-client-connection.md) joins, they will get the most recent call of the method with the data within the parameters.
* _**RunLocally**_ - If set to true, the caller will run the method logic locally, avoiding the networking route. If the server calls it, the server will also run the method. If a client calls it, the client will run the method immediately, instead of awaiting the servers call.

#### TargetRPC:

The TargetRPC will call **Server -> Client**, or **Client -> Client** if your [network rules](../network-manager/network-rules.md) allow it. This means that the method called, will only be triggered on the client which [PlayerID](../playerid-client-connection.md) is given.

**It holds the following parameters:**

* _**RequireServer**_ - This will override any settings within the [Network Rules](../network-manager/network-rules.md) as to whether clients can call the ObserversRpc directly.
* _**BufferLast**_ - If set to true, when the target [client](../playerid-client-connection.md) joins again (we can only hold their data if we've seen them before), they will get the most recent call of the method with the data within the parameters.
* _**RunLocally**_ - If set to true, the caller will run the method logic locally, avoiding the networking route. If the server calls it, the server will also run the method. If a client calls it, the client will run the method immediately, instead of awaiting the servers call.

### RPC Info

RPC Info is a super useful tool to get information about an RPC that has just been sent.

You simply add the RPCInfo as a parameter of your RPC and default the value to **default** and it will auto populate upon receiving the RPC

```csharp
[ServerRpc]
private void TestRpc(int myValue, RPCInfo info = default)
{
    //Here we now have the RPC info
    Debug.Log($"Sender: {info.sender}")
}
```

This value is mostly used to get the sender.
