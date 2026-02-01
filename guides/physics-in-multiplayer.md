# Physics in Multiplayer

Networking physics is a tricky thing. Things need to align, collisions should be possible from all players and misalignment or lag are bound to happen with high ping.

There are essentially 3 main ways you can get physics to work with PurrNet. Which one is best, entirely depends on your use case and experience as a developer!

<table><thead><tr><th width="123">Method</th><th>Network Rigidbody</th><th>Input Sync</th><th>PurrDiction (CSP)</th></tr></thead><tbody><tr><td><a data-footnote-ref href="#user-content-fn-1">Workflow</a></td><td>Plug n play</td><td>Very easy</td><td>Hard</td></tr><tr><td><a data-footnote-ref href="#user-content-fn-2">Responsive</a></td><td>✅</td><td>❌</td><td>✅</td></tr><tr><td><a data-footnote-ref href="#user-content-fn-3">Competitive</a></td><td>❌</td><td>✅</td><td>✅</td></tr><tr><td><a data-footnote-ref href="#user-content-fn-4">Result quality</a></td><td><a data-footnote-ref href="#user-content-fn-5">4/5</a></td><td><a data-footnote-ref href="#user-content-fn-6">5/5</a></td><td><a data-footnote-ref href="#user-content-fn-7">4/5</a></td></tr></tbody></table>

As you can see from the matrix above, there isn't one solution that has everything. And of course there are more depth to the answers, which you can read below.

### [Network Rigidbody](../plug-n-play-components/network-rigidbody.md)

The [NetworkRigidbody](../plug-n-play-components/network-rigidbody.md) component is entirely plug-n-play. It'll do it's very best to try and synchronize physics forces over the network, so locally everything should feel responsive to all players. The issue is mis-alignments. Given the unpredictive nature of non-deterministic physics together with players with ping and varying input, the results are not highly precise.&#x20;

This means that it's a great option for games where high precision doesn't matter, as it'll feel good locally, and especially interactions with world objects will be responsive. However, if precision in your physics matter a lot, it'll not be the right option for you.

### [Input Sync](../systems-and-modules/network-modules/sync-types/syncinput.md)

This is the absolute easiest way to get physics over the network that'll look perfect and be 100% precise as well. On top of that, PurrNet has a [SyncInput](../systems-and-modules/network-modules/sync-types/syncinput.md) module which handles most for you, and even allows you to emulate ping on the host for maximum fairness even in a peer to peer scenario.

This option won't fail you. Where it does fall flat is on the responsive side. The nature of input sync is simple. Essentially the entire physics side of your game will be handled by the server, and players are essentially just watching the replay live. This means that it's very prone to input lag. But results will be as good as a local game!

This is generally the best option if you have a heavy controller type, like a rolling ball or a character slipping on ice for example. Because it gives you an easy way to mitigate the input lag using polish such as animations that can be played locally to give the illusion of instant input, whilst the actual simulation is ran on the server.

### [PurrDiction](../client-side-prediction/) (Client Side Prediction)

With PurrNet, we have our own implementation of the technique called client-side-prediction. An implementation we're very proud of. In short: CSP is fully server auth similar to Input Sync, with the difference being that the clients are attempting to predict what's going to happen on the server. This utilizes input extrapolation with visual interpolation to yield the best possible results.&#x20;

This will not be a 100% perfect output visually, but it will be perfect most of the time, and even when imperfections happen, it'll smoothly solve it. It yields instant input response, which makes it feel great as well, and given it's a server authorized simulation, it's also safe for competitive play.

It can almost be seen as the mix between the Network Rigidbody and Input Sync, attempting to get the best of both worlds.

It is very important to note that PurrDiction is essentially it's own networking solution that does work with PurrNet, but it's a different way of writing logic and working with multiplayer. And mixing "normal" networking and prediction, can be a sticky affair if you don't know how the two systems interact.

[^1]: How easy it is to work with

[^2]: Whether it has input lag

[^3]: Can be used for competitive fairness

[^4]: How consistent will the results match across all screens

[^5]: This will depend on what you're doing and ping. It has adaptive features which should allow a lot of control over which cases are corrected harshly and which aren't.

[^6]: Regardless of ping, it'll visually align. but become more sensitive on the responsive front.

[^7]: Depends on ping, but it'll stay responsive regardless.
