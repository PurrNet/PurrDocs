# Awaitable RPC

Awaitable RPCs is a gamechanger for networking and optimal development properly integrating with the natural workflow of C#

{% embed url="https://youtu.be/UbcrjcE_DNo?si=UMqMJC9ZR5xwKBPR&t=198" %}

They utilize Tasks and also support UniTask. optionally they can be made async as well.

It not only allows for you to wait till RPC logic is finished, but allows you to return values from the receiving end of the RPC.

Below is a real use-case example of a way for the player to set ready on the server and it will return true if all players are ready, and false if not all players are ready (handled by the CheckForReady method)

It also has a fail safe in with the [networked State Machine](../../plug-n-play-components/state-machine-auto-networked.md) setup in case this isn't the game state were in:

```csharp
//The asyncTimeoutInSec is how long the caller should await a response. 
//In this case it would fail after 10 seconds (default: 5)
[ServerRpc(requireOwnership:false, asyncTimeoutInSec:10)]
public async Task<bool> SetReady(RPCInfo info = default)
{
    if (machine.currentStateNode != this) 
        return false;
    
    if(!_readyPlayers.Contains(info.sender))
        _readyPlayers.Add(info.sender);

    return CheckForReady();
}
```

And for the client all that needs to be done is have an async method which can await for the result of the RPC. You can read more about the[ async keyword here](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async). Essentially it will allow the method to run asynchronous of your otherwise fully sequential code, allowing it to halt execution until it has received a response.

```csharp
private async void OnReady()
{
    var allReady = await shopPhase.SetReady();

    //If everyone is ready, the server handles switching state and we don't do any more
    if (allReady)
        return;
    
    ShowWaitingScreen();
}
```

### Bare bones example

Here is the bare minimum of using this setup

```csharp
private async void MyLocalMethod()
{
    var myBool = await TestBool();

    if (!myBool)
        return;
    
    Debug.Log("Yay we made it!");
}

[ServerRpc(requireOwnership:false)]
public async Task<bool> TestBool()
{
    return true;
}
```
