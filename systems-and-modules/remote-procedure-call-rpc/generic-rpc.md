---
description: >-
  Generic RPC's will allow you to easily send any data type over the network!
  You can even send multiple generics
---

# Generic RPC

Generics in C# are awesome for creating modular and scalable systems, and we didn't want PurrNet holding you back.

{% embed url="https://youtu.be/UbcrjcE_DNo?si=YGN-tsZxVZ03RCQR&t=104" %}

This works with all RPC types such as: **ServerRpc, ObserversRpC, TargetRPC**. It also works with [static ](static-rpc.md)and [awaitable ](awaitable-rpc.md)RPC's!

Below is a super simple example usage of generics with RPC's. It's as easy as that!

```csharp
private void SendFirst()
{
    TestRpc("Purrfect!", true);
}

private void SendSecond()
{
    TestRpc(69, 4.20f);
}

[ServerRpc]
private void TestRpc<T, P>(T myData1, P myData2)
{
    Debug.Log($"Received {myData1} with type of: {myData1.GetType()}");
    Debug.Log($"Received {myData2} with type of: {myData2.GetType()}");
}
```
