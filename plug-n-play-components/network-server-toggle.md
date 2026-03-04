---
icon: server
---

# Network Server Toggle

This component lets you activate or deactivate GameObjects and Behaviours based on whether the current instance is running as a server or a client. It works similarly to [Network Ownership Toggle](network-ownership-toggle.md), but instead of checking ownership, it checks the server/client role.

This is handy for things like hiding server-only debug visuals from clients, or disabling server-side logic components on the client build.

## Setup

1. Add a `NetworkServerToggle` component to your networked GameObject.
2. In the inspector, assign the GameObjects and Behaviours you want to control.

## Inspector Fields

| Field | Description |
| --- | --- |
| To Activate | GameObjects that will be **active on the server** and **inactive on clients**. |
| To Deactivate | GameObjects that will be **inactive on the server** and **active on clients**. |
| To Enable | Behaviours that will be **enabled on the server** and **disabled on clients**. |
| To Disable | Behaviours that will be **disabled on the server** and **enabled on clients**. |

## Example Use Cases

- Show a debug collider visualization only on the server
- Disable AI controller scripts on client builds
- Hide server-only UI panels (admin tools, server stats) from players
- Enable client-only cosmetic effects that the server doesn't need to process

The toggle runs automatically when the object spawns, so no code is needed. If you do need to trigger it manually (for example after a role change), you can call:

```csharp
networkServerToggle.Setup(asServer: true);
```
