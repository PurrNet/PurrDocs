# Don't destroy on load

Some networked objects need to persist across scene changes. The Network Manager itself is a good example, but this also applies to things like a player's persistent inventory or a global chat system. If these get destroyed when a new scene loads, you lose the connection and all the state that came with it.

You can move Network Identities to DDOL (Dont Destroy On Load) by simply doing it as you would normally, but it has to be prior to the network starting. So calling it in awake is best.

```csharp
private void Awake() 
{
    DontDestroyOnLoad(this);
}
```
