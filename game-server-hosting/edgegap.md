---
description: In partnership with EdgeGap - Simplifying game server deployment
---

# EdgeGap

In this simple guide, we'll go through how you get started hosting your dedicated server(s) with EdgeGap in PurrNet. Luckily, this is very easy!

First, start by getting setup and logged in to [EdgeGap](https://app.edgegap.com/) so you can access the dashboard. And after this, have a brief look at [their own documentation](https://docs.edgegap.com/), to get setup with their Unity editor tool.

**\[Insert video here going through using and deploying with EdgeGap in PurrNet]**

### Installing EdgeGap in Unity

Go your your top toolbar: `Tools -> PurrNet -> Addon Library -> Install EdgeGap`. This will automatically import the tool into your project!

{% hint style="warning" %}
**You must have** [**Git installed**](https://git-scm.com/downloads) on your system for Unity to fetch packages via git URLs.
{% endhint %}

<figure><img src="../.gitbook/assets/Screenshot 2025-10-14 213647.png" alt=""><figcaption></figcaption></figure>

To help you get started we have setup a menu item that allows you to verify all the required dependencies and also help guide you to properly install them or remind you to open required tools.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

But to give you a summary you need these tools:

* [Docker ](https://www.docker.com/products/docker-desktop/)(Desktop or CLI)
  * Even after installed you need to make sure it is open when you want to upload a new server build
* Install Unity Linux Build Support through UnityHub

For more details checkout [Edegap's documentation](https://app.gitbook.com/u/2sVib8J3vKeGKmdmCC8sZomnEdi1).

### Setting up PurrNet

When Edgegap is properly installed a little box will show up for supported transport that let's you toggle on or off the auto configuration of these transports.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The auto configuration will detect when inside an Edgegap environment automatically and use environment variables to automatically setup the correct ports and even SSL settings on your behalf. The client still needs to properly configure these but you can rest assured that the server is handled.

In case you have multiple UDP ports you are using for one server you might want to disable this and manually handle your setup.

### Deploying the server

There are a few steps required to reach from nothing to a deployed server.

1. Unity Linux Build
2. Containerize The Build Files
   1. This is why you need docker installed, if you are not familiar with docker you can think of this step almost like zipping your project. I'm making a huge disservice to the technology behind the scenes but you don't need those details and if you want to you can learn more for yourself.
3. Upload container to Edgegap
4. Deploy your application!

Feel free to checkout [Edgegap's documentation](https://docs.edgegap.com/learn/unity-games/getting-started-with-servers) for proper detailed step by step on this but we will give you a quick getting started guide here too.

#### Speed running first server

First, you want to open Edgegap's editor window.

<div align="left"><figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure></div>

Then fill in your token:

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Once you have that setup you can skip most of those steps and go straight to step 5.\
In step 5 you need to choose your target application, a server image name (this can be whatever you want) and finally just press the `Rebuild from Source` button.

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Once it finishes you can either `Deploy` from the website dashboard or from step 6.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Every container you upload will have a version, by default it's the date that it was built at which you can use to select the latest.

Once you have an application that is deployed you can try to connect from your editor or a client build.\
On the website's dashboard you can manually find the information required to connect:

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

You would use the \`Host\` as your address and \`External\` port as your client port.

### Easy as that!

Now you know how to get a server up and running and connect to it!\
You can expand on this by using their [matchmaking service](https://docs.edgegap.com/learn/matchmaking/getting-started) to automatically create matches for your users.
