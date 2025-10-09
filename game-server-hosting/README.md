---
hidden: true
---

# ☁️ Game Server Hosting

A dedicated server is a separate application that runs your game's server logic without any client-side components like rendering, input handling, or UI. This approach provides better performance, security, and scalability for multiplayer games.

The cost? Well you need the server hosted somewhere. Whether that is self-hosting or utilizing a platform like [EdgeGap](edgegap.md) to more easily do so.

Unlike a [Host](../terminology/host.md) setup where one player acts as both server and client, a dedicated server:

* Runs independently without any player directly connected to the machine
* Handles only server-side game logic and networking
* Doesn't render graphics or process player input locally
* Can run on specialized server hardware or cloud platforms
* Provides consistent performance regardless of player connections

### Benefits of Dedicated Servers

#### Performance

* Server performance isn't affected by client-side rendering or input
* Can run on optimized server hardware
* Better tick rate consistency

#### Security

* Server authority is maintained on separate, controlled hardware
* Reduced risk of cheating through client manipulation
* Game state validation happens on trusted infrastructure

#### Scalability

* Multiple server instances can run simultaneously
* Players can connect from anywhere without relying on peer-to-peer connections
* Easier handling of player disconnections and reconnections

### Downsides?

Well it's more setup, and typically higher running-costs than something like a relay or direct LAN connection. Also, if you have game logic that for some reason relies on visuals, this could be more complicated to handle on a headless server, given that rendering isn't happening.

### Setting up a dedicated server with PurrNet

In general the logic is pretty straight forward. Choose your transport of choice, typically this would be the [UDP transport](../systems-and-modules/transports/udp-transport.md) or the [Web Transport](../systems-and-modules/transports/web-transport.md) depending on your game's target platform.

You spin up a dedicated server, and provide the transport layer with the IP and port. From here, you can make a headless build and deploy that to your server, and clients should now be able to connect.

PurrNet already has the auto start flags in the [network manager](../systems-and-modules/network-manager/). It's generally recommended setting this to start with a server build, to make sure the connection is being spun up automatically. If you have scene handling in your game, like starting in a main menu, you'll also need to redirect your server to move to the game scene for it to be able to run the connection.
