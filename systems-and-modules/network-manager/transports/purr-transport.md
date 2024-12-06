# Purr Transport

The Purr Transport is a relay provided by us to you. We wanted to allow the best developer experience and not having to setup servers or do port forwarding to test your game in an online scenario is a big pain point we are trying to take away from you.

The transport is in very early stages and will undergo multiple changes and you might experience server restarts once in a while. That being said it should be more than fine for testing purposes.

It works through a room system, you just need to specify a unique room and anyone can connect to it through the relay. There is still the notion of server and client and only one client can act as the host. Once the host disconnects everyone else is also kicked out similar to the other transports.

There are multiple regions available to you, right now when you create a room it will ping and get the one closest to you. We have 2 servers in the US, one in EU, one in AP, one in Brazil and one in South-Africa. This should be enough coverage and if needed we might expand on these regions in the future.

To use this transport simply add the `PurrTransport` component and assign it to your `NetworkManager`.
