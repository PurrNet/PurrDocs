# Network Assets

The Network Assets is a scriptable object used by PurrNet to cross identify any object in your project. This is used for cross referencing during the reading and writing process, meaning anything in this scriptable can be utilized for syncing, RPC's and more!

This is set as a reference on your [NetworkManager](./)

{% embed url="https://youtu.be/fj61Q8NEL9o" %}

Settings of the Network Assets scriptable allows you to easily modify which project objects/assets it will pull when you hit the generate button.

Refreshing the type list might be necessary if you add or remove types from your game, like making a new scriptable object for example.

### How to use it?

All you need is the network assets assigned in your network manager and do networking as you normally would and it will just work!\
(Make sure your asset is in the assets list)

### Is this needed?

Absolutely not. This is an "opt-in" situation. You can utilize it for networking anything, but you don't have to, and you can still easily write your own serialization as well. This is used as a first layer of fallback, so if some type can't be written, it'll send it from the Network Assets Scriptable.
