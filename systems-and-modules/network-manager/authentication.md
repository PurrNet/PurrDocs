# Authentication

If anyone can connect to your game server without verification, you have no way to prevent unauthorized players from joining, and no way to identify who is who. Authentication gives you a gate at the connection level, so you can verify players before they get access to anything in your game. This could be as simple as a password check, or as complex as verifying a token from a backend service.

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
using PurrNet.Transports;

namespace YourNamespace
{
    // this attribute ensures the correct Type variant is registered
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

        protected override Task<AuthenticationResponse> ValidateClientPayload(Connection conn, string payload)
        {
            // the server will validate it and return the appropriate response
            bool isValid = _password == payload;
            return Task.FromResult(new AuthenticationResponse { success = isValid });
        }

        protected override void UnAuthenticateClient(Connection conn) { }
    }
}
```

### **3. Using the Custom Authenticator**

To use your custom authenticator add it somewhere on your scene as a component and link it to your network manager like so:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Congratulations! You've successfully created a custom authenticator in PurrNet. This allows you to control access to your multiplayer game more securely and flexibly.

## Sending denial reasons to the client

When a connection is rejected, you often want the client to know **why** so its UI can react ("wrong password", "server full", "banned until X"). PurrNet ships an opt-in second base class, `AuthenticationBehaviour<TRequest, TDenial>`, that adds a typed denial payload to the existing flow.

The original `AuthenticationBehaviour<T>` is unchanged — pick the typed-denial overload only when you need a structured rejection reason on the client.

### How it works on the wire

1. Server's `ValidateClientPayload` returns `success = false` with a populated `denialReason`.
2. PurrNet serializes `TDenial`, fires `onAuthenticationDenied` server-side, and ships an `AuthenticationDenialPacket` to the client.
3. The client's authenticator decodes the reason, raises `OnDeniedByServer` / `onDeniedByServer`, then sends an ACK back.
4. On ACK the server cleanly closes the connection. If the ACK never arrives, the server force-closes after the authenticator's `timeout` (5 s by default), so a misbehaving client can't pin a slot indefinitely.

### **1. Define your denial type**

`TDenial` must be a struct (so it can be wrapped in `Nullable<T>` at the API boundary). Either an enum or a small `IPackedAuto` struct works.

```csharp
using PurrNet.Packing;

public struct LoginDenial : IPackedAuto
{
    public DenialReasonCode code;
    public string message;
}

public enum DenialReasonCode : byte
{
    WrongPassword = 0,
    Banned = 1,
    ServerFull = 2,
}
```

### **2. Inherit from the typed-denial behaviour**

```csharp
using System.Threading.Tasks;
using UnityEngine;
using PurrNet.Authentication;
using PurrNet.Transports;

namespace YourNamespace
{
    [RegisterNetworkType(typeof(AuthenticationRequest<string>))]
    public class CustomAuthenticator : AuthenticationBehaviour<string, LoginDenial>
    {
        [SerializeField] private string _password = "YourSecretPassword";

        protected override Task<AuthenticationRequest<string>> GetClientPayload()
        {
            return Task.FromResult(new AuthenticationRequest<string>(_password));
        }

        protected override Task<AuthenticationResponse<LoginDenial>> ValidateClientPayload(
            Connection conn, string payload)
        {
            if (payload != _password)
            {
                return Task.FromResult(AuthenticationResponse<LoginDenial>.Deny(new LoginDenial
                {
                    code = DenialReasonCode.WrongPassword,
                    message = "Incorrect password."
                }));
            }

            return Task.FromResult(AuthenticationResponse<LoginDenial>.Accept());
        }

        // Convenience override — fires on the client for both built-in and authenticator denials.
        // 'reason' is null for built-in deniers; only AuthenticatorRejected carries a value.
        protected override void OnDeniedByServer(DenialKind kind, LoginDenial? reason)
        {
            if (reason.HasValue)
                Debug.Log($"Server rejected login: {reason.Value.code} - {reason.Value.message}");
            else
                Debug.Log($"Server rejected connection: {kind}");
        }

        protected override void UnAuthenticateClient(Connection conn) { }
    }
}
```

You can also subscribe to the public event from anywhere on the client (e.g. your UI):

```csharp
authenticator.onDeniedByServer += (kind, reason) =>
{
    // 'reason' is LoginDenial? — null for built-in deniers, populated for AuthenticatorRejected.
    if (reason.HasValue)
        ShowError(reason.Value.message);
    else
        ShowError($"Disconnected: {kind}");
};
```

### Built-in denial kinds

Every rejection — whether from your authenticator or from PurrNet itself — surfaces a `DenialKind`:

| Kind                  | When it fires                                                            | Carries a reason payload? |
| --------------------- | ------------------------------------------------------------------------ | ------------------------- |
| `VersionMismatch`     | Client/server `NetworkManager.version` differ and rules say `Deny`       | No                        |
| `Timeout`             | Client didn't complete the auth handshake within the authenticator's `timeout` | No                  |
| `AuthenticatorRejected` | Your `ValidateClientPayload` returned `success = false`                | Yes (your `TDenial`)      |
| `NoAuthenticator`     | Client sent a non-auth request while the server has an authenticator     | No                        |

### Listening server-side

`NetworkManager` exposes a server-side event so you don't need to fetch the auth module:

```csharp
networkManager.onAuthenticationDenied += (conn, kind, reason) =>
{
    // 'reason' is null for built-in deniers (no payload).
    // It only contains bytes when an AuthenticationBehaviour<TRequest, TDenial>
    // attached a typed denial — handlers don't need to decode it themselves.
    Debug.Log($"Denied {conn} ({kind})");
};
```

The event fires before the denial packet is sent to the client, so logging/metrics fire even if the client never acks.
