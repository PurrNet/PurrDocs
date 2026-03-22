---
icon: wifi
---

# Transports

Not every game connects players the same way. Some games run over direct UDP connections, others need WebSocket support for browser builds, and some use platform-specific systems like Steam's networking. The transport layer is what actually moves your data between machines, and being able to swap it out without changing any of your game code means you can support different platforms and connection methods easily.

Transports are what's used to transfer the actual data over the network. Your chosen transport has to be referenced in the Network Manager. The default transport of PurrNet is the [UDP Transport](udp-transport.md).

{% embed url="https://youtu.be/I-uCr3pIJvk?si=acGf9yY_qZzAjf42" %}
