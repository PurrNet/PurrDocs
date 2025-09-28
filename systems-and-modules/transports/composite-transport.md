# Composite Transport

The composite transport takes in multiple transports and will automatically pick the support transport. The easiest example is if you have the [Web Transport](web-transport.md) and the [UDP Transport](udp-transport.md), and upload the game to the web, it will automatically utilize the [Web Transport](web-transport.md).

This works by automatically picking the first supported transport layer in the list. This allows you to easily utilize multiplayer transport layers in your project without needing to manually manage anything.

This also allows for easy cross platform support, as the server spins up all possible transport layers, and clients will pick one to connect to. So you can have players playing together between web, Steam, Epic and more!
