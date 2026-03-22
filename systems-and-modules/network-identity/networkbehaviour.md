# NetworkBehaviour

In Unity, MonoBehaviour is the base class for any script that lives on a GameObject. But MonoBehaviour has no awareness of the network. It doesn't know if it's running on the server or a client, who owns the object, or how to send data to other machines. NetworkBehaviour adds all of that.

Network Behaviour should be treated like your MonoBehaviour but for acting on the network. This also inherits from [Network Identity](./). From your Network Behaviour scripts, you'll be able to call [RPC's](../remote-procedure-call-rpc/), handle [synchronization](../network-modules/sync-types/syncvar.md), transfer [ownership](ownership.md), and more.
