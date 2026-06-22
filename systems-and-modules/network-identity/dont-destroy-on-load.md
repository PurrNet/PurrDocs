# Don't destroy on load

Some networked objects need to persist across scene changes. The Network Manager itself is a good example, but this also applies to things like a player's persistent inventory or a global chat system. If these get destroyed when a new scene loads, you lose the connection and all the state that came with it.

You can move Network Identities to DDOL (Dont Destroy On Load) by simply doing it as you would normally, but it has to be prior to the network starting. So calling it in awake is best.

```csharp
private void Awake() 
{
    DontDestroyOnLoad(this);
}
```

Note that PurrNet does not track the DDOL scene by default unless the Network Manager itself is also located there. If your manager lives in a different scene, any Network Identities you move to DDOL will fail to register and will show up as "Not Spawned".

To fix this, locate your NetworkRules asset, go to the scene rules section, and enable `alwaysIncludeDontDestroyOnLoadScene`. This explicitly tells PurrNet to register the DDOL scene so it can discover and spawn your persistent objects correctly.
