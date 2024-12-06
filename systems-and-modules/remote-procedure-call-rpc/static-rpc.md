---
description: Static RPCs allow you to send data without a NetworkIdentity present!
---

# Static RPC

Utilizing statics are super common during your development flow, so why shouldn't you with networking as well!

{% embed url="https://youtu.be/UbcrjcE_DNo?si=QsgiW3M5Na--U9di&t=22" %}

Now keep in mind that this can be unsafe if you run gameplay logic through it, as we don't have a [network identity](../network-identity/) to check ownership with.

This works with all RPC types such as: **ServerRpc, ObserversRPC, TargetRPC**. It also works with [generic ](generic-rpc.md)and [awaitable ](awaitable-rpc.md)RPC's!

```csharp
private void SendData()
{
    TestRpc(123);
}

[ServerRpc]
private static void TestRpc(int myNumber)
{
    Debug.Log($"Received {myNumber}");
}
```
