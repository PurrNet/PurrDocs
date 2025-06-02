---
description: Broadcast and chat
---

# Chat system with broadcasts

## Introduction

Broadcasting in PurrNet is useful, as it allows us to do some basic network functionality without needing a [Network Behaviour](../systems-and-modules/network-identity/networkbehaviour.md) on our object. For things that are trivial, such as game chat, we don't necessarily need all the functionality of a [Network Behaviour](../systems-and-modules/network-identity/networkbehaviour.md).

The idea is as follows:

1. Create a `ChatMessage` struct to store data such as a `name` and `message`.
2. Send a `ChatMessage` with our desired `name` and `message` to the **Server**.
3. Receive `ChatMessage` on the **Server**, then broadcast the `ChatMessage` to all **Clients**.
4. Receive `ChatMessage` on the **Clients**, then print the `ChatMessage` out.

## Creating the `ChatMessage` struct:

To get our chat message to the **Server**, we need to first create a struct to hold our data. As previously mentioned, this struct will hold a `name`, and a `message`. This struct will need to implement the [IPackedAuto](../systems-and-modules/bitpacker-serialization/networking-custom-classes-structs-and-types.md#IPackedAuto) interface, which will automatically handle the reading and writing of the data to the network. If this is not your style, take a look at the [IPacked](../systems-and-modules/bitpacker-serialization/networking-custom-classes-structs-and-types.md#IPacked) and [IPackedSimple](../systems-and-modules/bitpacker-serialization/networking-custom-classes-structs-and-types.md#IPackedSimple) interfaces.

The final struct is as follows:

```csharp
public struct ChatMessage : IPackedAuto
{
    public string name;
    public string message;
}
```

## Sending a `ChatMessage` to the **Server**

For our **Clients** to be able to send a message to the **Server**, we first need to hook into the `Subscribe` event from the `NetworkManager`

```csharp
void NetworkManager.Subscribe<ChatMessage>(PlayerBroadcastDelegate<ChatMessage> callback, bool asServer)
```

The easiest way to do this, is create a script that inherits from `PurrMonoBehaviour`, as this gives us access to two very useful events:

```csharp
public abstract void Subscribe(NetworkManager manager, bool asServer);
public abstract void Unsubscribe(NetworkManager manager, bool asServer);
```

For our case, let's create a `ChatManager` script, that inherits from `PurrMonoBehaviour` and subscribe to our chat events, as well as Creating an `OnChatMessage` function to pass in as our callback:

```csharp
public class ChatManager : PurrMonoBehaviour
{
    // Subscribe to ChatMessage events as either the Server, Client, or both
    public override void Subscribe(NetworkManager manager, bool asServer)
    {
        manager.Subscribe<ChatMessage>(OnChatMessage, asServer);
    }

    // Unsubscribe to ChatMessage events as either the Server, Client, or both
    public override void Unsubscribe(NetworkManager manager, bool asServer)
    {
        manager.Unsubscribe<ChatMessage>(OnChatMessage, asServer);
    }

    // Called when a ChatMessage broadcast is sent from either the Server or a Client
    private void OnChatMessage(PlayerID player, ChatMessage data, bool asServer)
    {
        // TODO: Make this work
    }
}
```

Now that we've subscribed to the events required, we can actually send a `ChatMessage` to the **Server**! You can do this however you'd like, for testing, something like this will be more than sufficient for our needs:

```csharp
void Update()
{
    if (Keyboard.current.enterKey.wasPressedThisFrame)
    {
        ChatMessage message = new ChatMessage
        {
            name = InstanceHandler.NetworkManager.localPlayer.ToString(),
            message = "Hello World!"
        };

        InstanceHandler.NetworkManager.SendToServer<ChatMessage>(message);
    }
}
```

## Receiving a `ChatMessage` on the **Server**, and sending it to all **Clients**

Now that we are sending messages from the **Client**, let's update our `OnChatMessage` function to handle receiving a `ChatMessage` broadcast on the **Server**. As mentioned, if we are the **Server** receiving the broadcast, we want to relay this information and broadcast it back to all of our **Clients**, and we can do it very simply with `NetworkManager.SendToAll<ChatMessage>(ChatMessage)`:

```csharp
// Called when a ChatMessage broadcast is sent from either the Server or a Client
private void OnChatMessage(PlayerID player, ChatMessage data, bool asServer)
{
    if (asServer)   // The broadcast was sent to the Server from a Client
    {
        // Send the broadcast down to the Clients
        InstanceHandler.NetworkManager.SendToAll<ChatMessage>(data);
    }
}
```

## Receiving a `ChatMessage` on the **Client**

Now that we are receiving messages from the **Server**, we can use our same `OnChatMessage` function to handle the data from the **Server**. For now, let's just debug the message:

```csharp
// Called when a ChatMessage broadcast is sent from either the Server or a Client
private void OnChatMessage(PlayerID player, ChatMessage data, bool asServer)
{
    if (asServer)   // The broadcast was sent to the Server from a Client
    {
        // Send the broadcast down to the Clients
        InstanceHandler.NetworkManager.SendToAll<ChatMessage>(data);
    }
    else    // The broadcast was sent to the Clients from the Server
    {
        Debug.Log($"Received {data.message} from {data.name}!");
    }
}
```

## Wrap Up

With what we have, our final script should look like such:

```csharp
public struct ChatMessage : IPackedAuto
{
    public string name;
    public string message;
}

public class ChatManager : PurrMonoBehaviour
{
    void Update()
    {
        if (Keyboard.current.enterKey.wasPressedThisFrame)
        {
            ChatMessage message = new ChatMessage
            {
                name = InstanceHandler.NetworkManager.localPlayer.ToString(),
                message = "Hello World!"
            };

            InstanceHandler.NetworkManager.SendToServer<ChatMessage>(message);
        }
    }

    // Subscribe to ChatMessage events as either the Server, Client, or both
    public override void Subscribe(NetworkManager manager, bool asServer)
    {
        manager.Subscribe<ChatMessage>(OnChatMessage, asServer);
    }

    // Unsubscribe to ChatMessage events as either the Server, Client, or both
    public override void Unsubscribe(NetworkManager manager, bool asServer)
    {
        manager.Unsubscribe<ChatMessage>(OnChatMessage, asServer);
    }

    // Called when a ChatMessage broadcast is sent from either the Server or a Client
    private void OnChatMessage(PlayerID player, ChatMessage data, bool asServer)
    {
        if (asServer)   // The broadcast was sent to the Server from a Client
        {
            // Send the broadcast down to the Clients
            InstanceHandler.NetworkManager.SendToAll<ChatMessage>(data);
        }
        else    // The broadcast was sent to the Clients from the Server
        {
            Debug.Log($"Received {data.message} from {data.name}!");
        }
    }
}
```
