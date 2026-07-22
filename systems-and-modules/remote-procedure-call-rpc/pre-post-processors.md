# Pre/Post Processors

In the name of being modular, we made at your disposal some processors you can register to handle what happens before and after an RPC is sent. Here is what they look like:

```csharp
public class RPCModule
{
    public delegate void RPCPreProcessDelegate(RPCSignature signature, ref BitPacker packer);

    public delegate void RPCPostProcessDelegate(RPCInfo info, ref BitData packer);
    
    public static event RPCPreProcessDelegate onPreProcessRpc;
    
    public static event RPCPostProcessDelegate onPostProcessRpc;
}
```

These can be used to append, intercept, change or edit data coming in and out of RPC traffic. Internally we use this for compression, you may have other uses.
