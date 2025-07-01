# Lobby System

With PurrNet, it's exceptionally easy to get started with a lobby in your game. We've built a plug n' play lobby setup for you to use with any provider (or self host) that you'd like. It comes with a Steam provider ready to use.

{% embed url="https://www.youtube.com/watch?v=fIBAlOJxqtg" %}

### Importing

PurrNet has it's own Addon Library window, which you can use to import tools, systems, examples and more with just one click.\
Start by going to:\
**Tools -> PurrNet -> Addon Library**

Here you'll be able to find the **PurrNet Lobby** setup. Go ahead and hit install and import it using the Unity import window that will pop up.

### Setup

Open up your build settings, and add the two scenes found in the setup to your build settings. These scenes are found in the **PurrLobby -> LobbyScenes** folder. After doing this, open up the **LobbySample** scene, and if you haven't already, import the TMP essentials from the newly shown window.

### Choosing a provider

At the time of writing this guide, the only providers available is the[ **Steam Provider**](steam-provider.md) and the [**Unity Lobby Provider**](unity-lobby-provider.md)**.** The steam provider is already in the scene by default. If you want to use a different provider, you can go to the Making your own provider section to read more. Alternatively, more providers might be available already, or built by the community.

In order to choose a provider, all you need to do is open the **LobbyManager** gameobject in the hierarchy. Here you'll see the **Lobby Manager** component, which acts as the brain of the whole system. Then you have "providers" which are responsible for the communication between the brain and the database of your choice (ex. Steam, Unity Lobbies, Epic Games, Your own database, etc.)

You can change providers by having them on the **LobbyManager** gameobject as components. A dropdown will be at the top of the **Lobby Manager** component, allowing you to choose which provider to use.

### Lobby Manager

As mentioned in the [Choosing a provider](./#choosing-a-provider) section, the **Lobby Manager** acts as the brain of the system.

It's very simple to use, yet very powerful for both debugging and an easy setup. A few foldout groups can be found on this:

* Create Room Arguments\
  These are the arguments used when creating a new room. These can be overwritten through code, so what you see in the inspector, are simply defaults.\
  The Room Properties act as meta information stored in the lobbies (if the provider allows for it). This is also what will be used as parameters for the searching with the browser tool.
* Search Room Arguments\
  These are responsible for the "filters" used when browsing lobbies. If these don't align with anything that is in the properties when the room is created, it won't be able to find the room. Some default values are setup for you already.
* Lobby Room Status\
  This status view is very powerful, in order for you to see what the **Lobby Manager** sees of information. You can get a full overview of the lobby, including it's members and information of those.\
  This information will work regardless of which provider you choose to use!
