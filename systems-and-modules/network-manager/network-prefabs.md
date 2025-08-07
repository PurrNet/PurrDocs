# Network Prefabs

The Network Prefabs is a scriptable object used by PurrNet to cross identify prefabs. This is used for [spawning and despawning](../network-identity/spawning-and-despawning/) of objects.

This is set as a reference on your [NetworkManager](./)

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption><p>The Network Prefabs scriptable having pulled a bunch of project prefabs</p></figcaption></figure>

Settings of the Network Prefabs scriptable allows you to easily modify from where it pulls the objects. If you have a prefabs folder which you intend to hold all prefabs, it would be optimal to set the folder to that. It will automatically search any sub-folders.

Settings can also modify whether it should auto generate to find prefabs or you'd want to handle it manually, and lastly it can also find any non-networked objects and add them as well, so even objects without a network identity script, can be spawned over the network. This will automatically add the PrefabLink script to it.
