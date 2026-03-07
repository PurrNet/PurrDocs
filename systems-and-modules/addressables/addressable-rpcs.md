# Addressable RPC's

In order to make it exceptionally easy to work with Addressables, it's important to also make it easy to work with RPC's.

For this reason, the [Async packing](../bitpacker-serialization/async-serialization-packing.md) was added to PurrNet, and we've also built an out of the box networked type called `NetworkAddressable` to help you easily send assets over the network.

Here is an example usage. This works with any AssetReference, in this example a gameobject:

```csharp
public AssetReferenceGameObject prefabReference;

private void SendTestReference()
{
    //Implicit conversion to the NetworkAddressable:
    ReceiveAddressable(prefabReference);
}

[ObserversRpc]
private void ReceiveAddressable(NetworkAddressable addressable)
{
    //Implicit conversion back to a normal addressable
    Debug.Log($"Received addressable: {addressable}", addressable.Asset);
}
```

It handles implicit conversions for you, so as you can see in the example code, you just work with it as you would with any addressable.
