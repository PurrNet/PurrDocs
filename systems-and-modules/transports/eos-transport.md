# EOS Transport

The EOS Transport lets you use [Epic Online Services](https://dev.epicgames.com/en-US/services) as a transport layer for PurrNet. It runs P2P, with no relay to host through and no extra glue. Drop it on your `NetworkManager` and PurrNet rides on top of EOS.

This is the official PurrNet EOS Transport. There is also a [community version](eos-transport-community.md) maintained separately.

### Installation

The easiest way is to open **Tools / PurrNet / PurrNet Packages** and hit install on EOS Transport. One click and you're done.

If you'd rather pull it in by hand, open Unity's Package Manager, click **Add package from git URL**, and paste one of these.

Stable:

```
https://github.com/PurrNet/PurrNetEOSTransport.git?path=/Assets/EOSTransport#release
```

Latest in-development:

```
https://github.com/PurrNet/PurrNetEOSTransport.git?path=/Assets/EOSTransport#dev
```

You can also pin to a specific tag with `#v1.0.0` (or whatever the latest tag is) instead of a branch.

You'll also need the [PlayEveryWare EOS Plugin](https://github.com/PlayEveryWare/eos_plugin_for_unity_upm). Same flow, Package Manager, **Add package from git URL**:

```
https://github.com/PlayEveryWare/eos_plugin_for_unity_upm.git
```

That gives you the EOS SDK and `EOSManager`. The transport's runtime code is gated by a `versionDefine` on `com.playeveryware.eos`, so it only compiles once the plugin is in.

Then set up your Epic dev account and product credentials under **Tools / EOS Plugin / EOS Configuration**.

### Usage

1. Add the `EOSTransport` component to your `NetworkManager`.
2. Set `socketName` to any string. Both peers just need to agree on it.
3. On the client, set `remoteProductUserId` to the **host's** EOS Product User ID before connecting.
4. Call `NetworkManager.StartServer()` to host, `NetworkManager.StartClient()` to join.

If you call `StartClient()` on the same `NetworkManager` that's already hosting, it short-circuits to in-process delivery instead of going through EOS. Host loopback, no extra wiring.

*If you're testing with multiple unity projects and running into connection issues make sure to focus the server window after connecting a client or it will fail due to inactivity.* 

{% embed url="https://github.com/PurrNet/PurrNetEOSTransport" %}
