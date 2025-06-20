# Pooling

Pooling is the act of re-utilizing prefabs when old ones are no longer being used. In PurrNet you can use pooling with networked objects. The way it works is that when an object is despawned or you lost visibility of it, instead of `Destroying` it, we will store it behind the scenes until you try to spawn another of that thing.

To can enable pooling on your network prefabs:

<figure><img src="../../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

The way to utilize pooling is quite simple with PurrNet, in fact if you ticked the box you are already done. You keep calling `Instantiate` and `Destroy` and we handle it for you.

If you need to reset some data once it get pooled you have an override on your `NetworkIdentities` that gets called for cleanup:

```csharp
protected override void OnPoolReset()
{
    // reset some stuff
}
```

There is one caveat. PurrNet allows pooling of children separately and even partial pooling where a prefab might be constructed partly from pooled parts and partly from instantiated parts. But this does cause one issues which is possible mangling or loss of references. Note that this is only an issue when you are mix and matching prefab parts and parents. To fix this we made a `Reference<>` module available for you and it is meant to be used like this:

```csharp
[SerializedField] private Reference<MeshRenderer> someRenderer;
```

Now your `MeshRenderer` reference will be repaired if it ever breaks.
