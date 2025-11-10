# Installation

## Adding it to your project

You first need to make sure you have `PurrNet v1.14.0` or higher [installed ](../../getting-started/installation-setup.md)as PurrDiction uses PurrNet to function.

### **Git through Package Manager**

Through the package manager you can add a package using a git URL. Just add the following URL, and whenever you need PurrDiction updated, you can update it directly from the package manager:

```clike
https://github.com/PurrNet/PurrDiction.git?path=/Assets/PurrDiction#release
```

You can also use the `dev` branch if you're looking to access the latest features, though it may come at the cost of stability. You can switch at any time by using either URLs:&#x20;

```clike
https://github.com/PurrNet/PurrDiction.git?path=/Assets/PurrDiction#dev
```

#### ⚠️ Requirements

* **You must have Git installed** on your system for Unity to fetch packages via git URLs.
* If you just installed Git, **restart Unity and Unity Hub** before trying again.
* If it still doesn't work after restarting Unity, **restart your computer**.

### **OpenUPM**

We are also registered on OpenUPM if you prefer using it [![openupm](https://img.shields.io/npm/v/dev.purrnet.purrdiction?label=openupm\&registry_uri=https://package.openupm.com)](https://openupm.com/packages/dev.purrnet.purrdiction/)

```shell
openupm add dev.purrnet.purrdiction
```

### **Unity Asset store**

{% hint style="warning" %}
The asset is still in the process of being approved by the Unity team
{% endhint %}

It's as easy as going to the asset store using this link, and adding it like a normal package:

[https://assetstore.unity.com/packages/slug/329734](https://assetstore.unity.com/packages/slug/329734)

### **Package Import**&#x20;

You can download the latest release from [this page](https://github.com/PurrNet/PurrDiction/releases) and simply double click it to import it into your project
