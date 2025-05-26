# PlayerID (Client connection)

PlayerID is the struct which PurrNet uses to keep track of all clients that is currently or previously was connected to the server.

This data can remain persistent, depending on your settings on the Network Manager. If a client disconnect, and later reconnects, they will keep the same data such as player ID and meta information.

This makes it easy to identify players, implement saving/loading and much more.
