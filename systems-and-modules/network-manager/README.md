---
icon: brain
---

# Network Manager

The Network Manager is the very core of the network. It acts as the heart and brain of PurrNet, so all settings of your network are setup through your network manager.

Look at the [Installation/Setup](../../guides/installation-setup.md) segment for a guide on the setup.

{% embed url="https://youtu.be/CdkCjyjMF7w?si=VDKglsqtj5PzcQTi" %}

### Settings in the Network manager

**Start Server Flags: These** are the conditions for the server to automatically start upon seeing the NetworkManager

**Start Client Flags:** These are the conditions for the client to automatically start upon seeing the NetworkManager

**Cookie Scope**: PurrNet has cookie/caching for data, and this is where and how the cookies act within PurrNet.\
\- Live With Connection: Nothing is stored once the connection stops\
\- Live With Process: Nothing is stored once the game/process stops\
\- Store Persistently:  Cookies are stored in the player prefs of your system

**Transport:** This is what transport to use for the data sending. By default, you'd likely want to be using the UDP transport. You can just add the transport to the same gameobject and drag it to this field.

**Network Prefabs:** These are the [Network Prefabs](network-prefabs.md) to use. Just simply select the scriptable.

**Network Rules:** These are the [Network Rules](network-rules.md) to use with the Network Manager. Just simply select the scriptable you want to use. This can be overwritten on the individual [network identity](../network-identity/).

**Visibility Rules:** This is the [Visibility ](network-visibility/)rule-set to use. This will decipher the default rules for player to Network Identity relation. This can be overwritten on the individual [network identity](../network-identity/).
