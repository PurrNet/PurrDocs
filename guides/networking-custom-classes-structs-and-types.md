# Networking custom classes, structs & types

PurrNet auto networks structs, classes and interfaces for you, given they directly or indirectly consist of serializable data.

Networking custom classes and structs can be extremely useful to send over the network, for example by using basic [RPC's](../systems-and-modules/remote-procedure-call-rpc/).

{% embed url="https://youtu.be/Ck13MQuYBV0?si=AUuMKixhUCXphEkK" %}

### IPackedSimple

This allows you to handle the serialization yourself. This is the easiest way if you need to make your own serializer.\
An example of using this is below:

```csharp
public struct ManagedStruct : IPackedSimple
{
    public int myInt;
    public object data;
    
    public void Serialize(BitPacker packer)
    {
        Packer<int>.Serialize(packer, ref myInt);
        Packer<object>.Serialize(packer, ref data);
    }
}
```

### IPacked

This also allows you to handle the serialization yourself. Contrary to the previous method, this allows you to disconnect the reading and writing of data, for more special cases.\
An example of using this is below:

```csharp
public struct ManagedStruct : IPacked
{
    public int myInt;
    public object data;
    
    public void Write(BitPacker packer)
    {
        Packer<int>.Write(packer, myInt);
        Packer<object>.Write(packer, data);
    }
    
    public void Read(BitPacker packer)
    {
        Packer<int>.Read(packer, ref myInt);
        Packer<object>.Read(packer, ref data);
    }
}
```

### Static custom type

This is the most performant way of handling custom reading and writing of your data. The PurrNet serialization system will automatically find your static type.\
Below is an example of using this with a Vector2:

```csharp
public static CustomSerializerForExternalType
{
    public static void Write(this BitPacker packer, Vector2 value)
    {
        packer.Write(value.x);
        packer.Write(value.y);
    }
    
    public static void Read(this BitPacker packer, ref Vector2 value)
    {
        packer.Read(ref value.x);
        packer.Read(ref value.y);
    }
}
```

### Packing directly RPC

You can pack data directly and optimally using the BitPackerPool and simply sending the BitPacker directly through an RPC. This uses less memory and allows you to easily pack custom data on the fly.

```csharp
protected override void OnSpawned(bool asServer)
{
    if (!asServer)
    {
        using var writer = BitPackerPool.Get();
        
        Packer<string>.Write(writer, "Hello, server!");
        Packer<string>.Write(writer, "Maybe some audio data being pumped?");

        MyRPC(writer);
    }
}

[ServerRpc(requireOwnership: false)]
private void MyRPC(BitPacker data)
{
    using (data)
   {
        string message = default;
        string audioData = default;

        Packer<string>.Read(data, ref message);
        Packer<string>.Read(data, ref audioData);

        PurrLogger.Log(message); // Hello, server!
        PurrLogger.Log(audioData); // Maybe some audio data being pumped?
    }
}
```
