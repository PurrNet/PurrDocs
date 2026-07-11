# Installation and setup

PurrDiction is an addon to PurrNet. Install a compatible PurrNet version first; PurrDiction's package does not declare PurrNet as an automatic UPM dependency.

## Install from Git

In Unity, open **Window > Package Manager**, select **Add package from git URL**, and use the stable branch:

```text
https://github.com/PurrNet/PurrDiction.git?path=/Assets/PurrDiction#release
```

Use the development branch only when you intentionally need unreleased work:

```text
https://github.com/PurrNet/PurrDiction.git?path=/Assets/PurrDiction#dev
```

Git must be installed and available to Unity. Restart Unity and Unity Hub after installing Git.

## Install from OpenUPM

PurrDiction's package name is `dev.purrnet.purrdiction`:

```shell
openupm add dev.purrnet.purrdiction
```

You can also configure the OpenUPM scoped registry and add the package by name through Unity's Package Manager.

## Configure a scene

1. Add one **Prediction Manager** to each scene that owns a predicted world.
2. Select which **Built In Systems** the scene needs: hierarchy, players, time, random, and 2D or 3D physics.
3. If using Unity physics, enable the matching flag in **Physics Provider** as well as the matching built-in physics system.
4. Choose whether view updates run in `Update`, `LateUpdate`, or not at all with **Update View Mode**.
5. Create a **Predicted Prefabs** asset from **Assets > Create > PurrNet > Purrdiction > PredictedPrefabs** and assign it to the manager when the hierarchy will create predicted prefabs.

The Predicted Prefabs asset can scan a selected folder, or all of `Assets`, for prefabs containing a `PredictedIdentity`. Each entry can optionally use pooling and a warmup count.

{% hint style="warning" %}
Both **Built In Systems > Physics 3D/2D** and the corresponding **Physics Provider** flag are required for PurrDiction-managed Unity physics. Do not drive predicted physics from `FixedUpdate`; the Prediction Manager advances physics during its tick simulation.
{% endhint %}

## First verification

Enter Play Mode as host and confirm that:

* The Prediction Manager spawns without registration errors.
* Any predicted prefab used by `PredictedHierarchy.Create` appears in the assigned Predicted Prefabs asset.
* Input and simulation callbacks run at the PurrNet tick rate.
* A remote client receives verified frames and corrects after simulated latency or packet loss.

After setup, continue with [Predicted Identities](predicted-identities.md) or the [built-in components](built-in-components/README.md).
