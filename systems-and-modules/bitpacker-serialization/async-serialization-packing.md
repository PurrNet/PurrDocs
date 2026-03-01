# Async Serialization (packing)

PurrNet supports asynchronous serialization, allowing you to pack and unpack data fully async for RPC utilization. This is particularly useful for workflows sensitive to timing, such as working with Addressables, databases, or anything requiring data to be fetched from outside of the project at runtime.

#### IAsyncPackable

To make a type async-packable, implement the `IAsyncPackable` interface. It exposes two lifecycle methods that PurrNet calls automatically around serialization:

| Method                  | When it's called                              |
| ----------------------- | --------------------------------------------- |
| PrepareForPackAsync     | Before the data is packed and sent            |
| PrepareAfterUnpackAsync | After the data has been received and unpacked |

These methods return `Task<IAsyncPackable>`, so you can `await` anything inside them — asset loading, database calls, scene queries, etc.

#### \[DontPack]

Fields marked with `[DontPack]` are excluded from serialization entirely. Use this for references that can't or shouldn't be sent over the network, and instead resolve them inside `PrepareAfterUnpackAsync`.

#### Example

The following example sends a `Renderer` reference across the network. Since `Renderer` isn't directly serializable, the struct stores the GameObject's name as a string, then resolves the renderer by name on the receiving end.

```csharp
private struct NetRenderer : IAsyncPackable
{
    public string goName;
    [DontPack] public Renderer renderer;

    public async Task<IAsyncPackable> PrepareForPackAsync()
    {
        if (!renderer)
        {
            Debug.LogWarning("PrepareForPackAsync: renderer is null");
            return this;
        }
        await Task.Delay(1000);
        goName = renderer.gameObject.name;
        return this;
    }

    public async Task<IAsyncPackable> PrepareAfterUnpackAsync()
    {
        await Task.Delay(1000);
        var go = string.IsNullOrEmpty(goName) ? null : GameObject.Find(goName);
        if (!go)
        {
            Debug.LogError($"No GO found by name: '{goName}'");
            return this;
        }
        renderer = go.GetComponent<Renderer>();
        return this;
    }
}
```

**What's happening here:**

* **Sender** — `PrepareForPackAsync` runs before the RPC is sent. It stores the GameObject's name into `goName`, which gets packed and transmitted.
* **Receiver** — `PrepareAfterUnpackAsync` runs after unpacking. It uses `goName` to find the GameObject in the scene and resolves the local `Renderer` reference.
* The `[DontPack]` attribute on `renderer` ensures the non-serializable component is never included in the packet itself.

To put very simply, since the renderer isn't serializable, we utilize the \[DontPack] attribute, to instruct the [bitpacker](./) to avoid packing this piece of data. And then we can simple get it using the name of the gameobject that's sent. This is obviously a quite silly use-case that doesn't have a big need for async code. But hopefully you can see how this is applicable in many scenarios where async can be helpful!
