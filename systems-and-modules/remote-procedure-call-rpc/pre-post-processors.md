# Pre/Post Processors

In the name of being modular, we made at your disposal some processors you can register to handle what happens before and after an RPC is sent. Here is what they look like:

<pre class="language-csharp"><code class="lang-csharp">public class RPCModule
{
<strong>    public delegate void RPCPreProcessDelegate(ref ByteData rpcData, RPCSignature signature, ref BitPacker packer);
</strong>    
    public delegate void RPCPostProcessDelegate(ByteData rpcData, RPCInfo info, ref BitPacker packer);
    
    public static event RPCPreProcessDelegate onPreProcessRpc;
    
    public static event RPCPostProcessDelegate onPostProcessRpc;
}
</code></pre>

These can be used to append, intercept, change or edit data coming in and out of RPC traffic. Internally we use this for compression, you may have other uses.
