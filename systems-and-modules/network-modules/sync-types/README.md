# Sync Types

All sync types (e.g., `SyncVar`, `SyncList`) are built on our network module system.\
Any limitations or features of `NetworkModule` apply to them as well.

This means you can create your own sync types or modules—`SyncVars` are just network modules, not magic. Anyone, including you, can build custom modules using the same system.

### What to expect of Sync types

There are some default expected behaviour of anything called a "Sync"-anything.&#x20;

* Live syncing: Any change done now, will be pushed to all players
* Buffering: When a new player joins mid-session (Late Join), they don't need to wait for the next change.

Of course bare in mind that there could be exceptions depending on the individual implementation.
