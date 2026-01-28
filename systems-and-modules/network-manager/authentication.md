# Authentication

In PurrNet, custom authenticators enable you to define your own authentication logic for multiplayer games. This guide will walk you through the steps to create a custom authenticator using a simple password-based example.

{% hint style="info" %}
Any new connections will never be promoted to `PlayerID` unless they have authenticated. They won't spawn any objects, receive any RPCs or player scoped Broadcasts, they will only receive `Connection` scoped broadcasts.
{% endhint %}

### **1. Create a script for your Authenticator**

Create a new C# script in your Unity project, and name it something meaningful, such as `CustomAuthenticator`.

### **2. Implement the Custom Authenticator Class**

Your custom authenticator class should inherit from `AuthenticationBehaviour<T>`. This ensures that your class adheres to the PurrNet authentication structure.

```csharp
using System.Threading.Tasks;
using UnityEngine;
using PurrNet.Authentication;

namespace YourNamespace
{
    // this attributes ensures the correct Type variant is registered
    [RegisterNetworkType(typeof(AuthenticationRequest<string>))]
    public class CustomAuthenticator : AuthenticationBehaviour<string>
    {
        [Tooltip("The password required to authenticate the client.")]
        [SerializeField]
        private string _password = "YourSecretPassword";

        protected override Task<AuthenticationRequest<string>> GetClientPayload()
        {
            // the client will send his password to the server
            return Task.FromResult(new AuthenticationRequest<string>(_password));
        }

        protected override Task<AuthenticationResponse> ValidateClientPayload(string payload)
        {
            // the server will validate it and return the appropriate response
            bool isValid = _password == payload;
            return Task.FromResult(new AuthenticationResponse(isValid));
        }
    }
}
```

### **3. Using the Custom Authenticator**

To use your custom authenticator add it somewhere on your scene as a component and link it to your network manager like so:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Congratulations! You've successfully created a custom authenticator in PurrNet. This allows you to control access to your multiplayer game more securely and flexibly. Experiment with different types of authenticators and tailor them to fit your specific requirements.
