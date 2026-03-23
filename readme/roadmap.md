---
description: >-
  An overview of what we're working on and what's coming next, roughly in
  priority order from top to bottom.
---

# 🗺️ Roadmap

* [ ] PurrLobby V2
* [ ] Network LOD
* [ ] PurrDiction sample
* [ ] Host migration
* [ ] Client owned scenes
* [ ] PurrVoice (WWise support)
* [ ] Deterministic physics for prediction
* [ ] Network Transform V2
* [x] [Network visualization tooling](#user-content-fn-1)[^1]
* [x] [Network Pipe logic (local transporting layer to components)](#user-content-fn-2)[^2]
* [x] PurrVoice (FMOD support)
* [x] Easy network conversion tools (Components)
* [x] Easy network conversion tools (Code)
* [x] [Full game sample](../full-game-guides/incremental-game-sample.md) (w. source code)
* [x] [Addressables](../systems-and-modules/addressables/) support (scenes, objects, RPC's)
* [x] [Async serialization](../systems-and-modules/bitpacker-serialization/async-serialization-packing.md) (async packing)
* [x] Epic Games Transport (EOS)
* [x] [Network Any Asset](../systems-and-modules/network-manager/network-assets.md)
* [x] PurrNet Plug-n-play [**voice chat**](../tools/purrvoice-voice-chat/)
* [x] Plug-n-play prediction components
* [x] Compression of prediction
* [x] Modularized delta compression
* [x] [Code-stripping](../systems-and-modules/code-stripping.md)
* [x] [Network Profiler tool](../systems-and-modules/bandwidth-profiler.md)
* [x] Improved logging
* [x] Lag compensation component ([collider rollback](../systems-and-modules/collider-rollback.md))
* [x] [Predicted state machine](../client-side-prediction/predicted-state-machine/)
* [x] [Client Side Prediction](../client-side-prediction/)
* [x] Delta serialization
* [x] Easy packet compression
* [x] [Authentication](../systems-and-modules/network-manager/authentication.md)
* [x] Plug n' play [lobby setup](../addons/lobby-system.md)
* [x] [Free relay](../systems-and-modules/transports/purr-transport.md) for easy development testing
* [x] Auto network [object pooling](../systems-and-modules/network-identity/pooling.md)
* [x] Nesting network modules
* [x] [Steamworks Transport](../systems-and-modules/transports/steam-transport.md)
* [x] Auto [networked state machine](../plug-n-play-components/state-machine-auto-networked.md)
* [x] [Auto Sync Component](../plug-n-play-components/network-reflection-auto-sync.md)
* [x] [Returnable/Awaitable RPCs](../systems-and-modules/remote-procedure-call-rpc/awaitable-rpc.md)
* [x] [SyncList](../systems-and-modules/network-modules/sync-types/synclist.md) & [SyncDictionary ](../systems-and-modules/network-modules/sync-types/syncdictionary.md)& [SyncHashset](../systems-and-modules/network-modules/sync-types/synchashset.md)
* [x] [SyncTimer](../systems-and-modules/network-modules/sync-types/synctimer.md)
* [x] [SyncEvent ](../systems-and-modules/network-modules/sync-types/syncevent.md)(Auto network invoked events)
* [x] Auto serialization
* [x] Awaitable Coroutines

[^1]: In scene visualization for things such as data usage heatmaps, or clear ownership visualization

[^2]: Allowing individual components to move data through unique transport layers for much more flexibility and scalability
