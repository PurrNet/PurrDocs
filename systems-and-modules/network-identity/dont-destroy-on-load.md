# Don't destroy on load

You can move Network Identities to DDOL (Dont Destroy On Load) by simply doing it as you would normally, but it has to be prior to the network starting. So calling it in awake is best.

```csharp
private void Awake() 
{
    DontDestroyOnLoad(this);
}
```
