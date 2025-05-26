---
icon: diamonds-4
---

# Instance Handler

The Instance Handler allows you to easily get access the [Network Manager](network-manager/) from anywhere, even non [Network Identity ](network-identity/)or [Network Behaviour ](network-identity/networkbehaviour.md)scripts:

<pre class="language-csharp"><code class="lang-csharp"><strong>var nm = InstanceHandler.NetworkManager;
</strong></code></pre>

But more importantly, it allows you to easily register, unregister and get instances yourself!\
Below is an example of registering and unregistering the instance, as well as the instances being used from static methods. Mind that  you can also get the instances from other scripts, whether they are MonoBehaviour or not.

```csharp
public class GameManager : NetworkBehaviour
{
    private void Awake() 
    {
        //We register the GameManager instance
        InstanceHandler.RegisterInstance(this);
    }
    
    private void OnDestroy() 
    {
        //Upon being destroyed, we unregister the game manager instance
        InstanceHandler.UnregisterInstance<GameManager>();
    }

    private static void GetInstanceExample() 
    {
        //This will fail if the manager isn't registered
        InstanceHandler.GetInstance<GameManager>().Success();
    }
    
    private static void  TryGetInstanceExample() 
    {
        //This will only run if we get the manager. Potentially invert the if statement and log and error if you don't get it
        if(InstanceHandler.TryGetInstance(out GameManager manager))
            manager.Success();
    }

    private void Success() 
    {
        Debug.Log("Now we're logging from the GameManager instance!", this);
    }
}
```
