# SyncTextureFile



<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Defining the module:

```csharp
[SerializeField] private SyncTextureFile _avatar;
```

Setting data as the controller:

```csharp
_avatar.filePath = _path;
```

Reacting to new data received:

```csharp
private void OnEnable()
{
    _avatar.onDataChanged += OnAvatarChanged;
}

private void OnDisable()
{
    _avatar.onDataChanged -= OnAvatarChanged;
}

private void OnAvatarChanged(Texture2D texture)
{
    _avatarImage.texture = texture;
}
```

