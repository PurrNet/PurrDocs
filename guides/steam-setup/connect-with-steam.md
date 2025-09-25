# Connect with Steam Using the Steam Transport
Greetings!

So you want to make a multiplayer Steam game, and you're looking to get started with PurrNet? You've come to the right place! My name is Judsin, and I'm going to show you how to connect with your friends over Steam so that you can get started testing today! I will do my best to be extremely clear and provide you with screenshots every step of the way. Before we begin, let's clarify what this document is and is not below:

#### This is:
✔️ A guide on how to use the Steam Transport\
✔️ A resource for how you can leverage PurrNet and connect with your friends over steam

#### This is not:
❌ A guide on how to use a lobby\
❌ A guide on how to use Steamworks.NET\
❌ A guide on how to test steam connections on a single machine

**Additional Remarks:** As a solo developer, it is often challenging to test on Steam as it requires two Steam accounts. There are many different ways to do this solo, but for the sake of this guide, I am going to assume that you have a friend or teammate that can help you test.

## Getting Started
This guide is being written using the following:
- Unity 6002.2.0f1
- PurrNet 1.15.0
- Heathen's Toolkit for Steakworks 2025
- Steam installed and running

For PurrNet installation instructions please reference the [installation guide](../../getting-started/installation-setup.md). If you aren't familiar with Heathen's Toolkit for Steakworks SDK I highly recommend checking it out if you're serious about using Steam—it just makes your life easier. If you don't want to use Heathen's, then the script example I show won't be relevant to you, but the majority of this guide will still be helpful. We will install Heathen's Toolkit at the end of this guide to ensure that everyone can follow along until then.

For reference, my Package Manager looks like the following after PurrNet installation:
![img.png](img.png)

### Step 1 Create Scene:
This is a fresh project, so I'm going to start by creating a new scene and putting it in the Assets/Scenes folder. I will call this scene "NetworkedSteamScene" and I will delete the "SampleScene" already in the folder:
![img_1.png](img_1.png)

Open the scene:

![img_2.png](img_2.png)

### Step 2 Create NetworkManager:
Right-click anywhere in the Hierarchy and go to PurrNet → NetworkManager
![img_3.png](img_3.png)

### Step 3 Configure GameObject:
Click on the newly created PurrNet GameObject and feel free to name it whatever you want; I prefer "NetworkManager":
![img_4.png](img_4.png)

Remove the UDP Transport and add the Steam Transport:
![img_5.png](img_5.png)

If you want to know more about the Steam Transport, please reference [Steam Transport](../../systems-and-modules/transports/steam-transport.md).

Now click "Add SteamWorks.Net to Package Manager" on the Steam Transport component—this will add Steamworks.NET to your project. If this gives you an error, please make sure you have git installed on your machine.

If this works correctly, then your setup should look like this:
![img_6.png](img_6.png)

Let's add the [Network Rules](../../systems-and-modules/network-manager/network-rules.md) to the Network Manager component by clicking the target icon and add the default rules:
![img_7.png](img_7.png)

Select the "Unsafe" rules for now if you don't know what these are and just want to get Steam working. Doing so will make the NetworkManager look like this:
![img_8.png](img_8.png)

Let's configure our NetworkManager to look like this:
![img_9.png](img_9.png)

### Step 4 Script Setup:
This step is where we will use Heathen's Toolkit for Steamworks to make our lives easier. If you don't have this package, the code below will still be useful to you. I will comment what is a Heathen's API call so that you know what parts you need to go solve with whatever Steamworks API wrapper you're using. Now let's install Heathen's Toolkit for Steamworks. Below is a screenshot of my PackageManager:
![img_13.png](img_13.png)

First things first we need to create a Steam Settings object. You can skip this part if you don't have Heathens. Below is a copy paste from the Heathen's website on how to do this:

Open your Project Settings and select Player > Steamworks

<figure><img src="https://3622319385-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FkfE3ZAs6TeNW5sBM2C3i%2Fuploads%2F2sgOBIYmVwUNtczwf4GS%2Fimage.png?alt=media&#x26;token=d9f90e0a-56ce-418b-b5b4-c334cd8e1d6d" alt=""><figcaption></figcaption></figure>

When you first do this it will create a SteamMain Steam Settings object in your project's Settings folder.

<figure><img src="https://3622319385-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FkfE3ZAs6TeNW5sBM2C3i%2Fuploads%2FFTVxnO4siYbyDIJ9BLaK%2Fimage.png?alt=media&#x26;token=0f3313b0-5e0f-4e2a-9ac1-c741a712237d" alt=""><figcaption></figcaption></figure>

You can optionally add a Demo setting and as many Playtest settings as you would like. All of them will be added to the Settings folder in their own Steam Settings object.

Next, let's add a canvas with a Host and Join button as well as a text field to display the Host ID (that only the host will see) and a text input field that the client can type in. This prompted me to install TextMesh Pro so I installed it:
![img_16.png](img_16.png)

Next, let's create a script to handle the connection to Steam. Create a new script called "ConnectionManager" and put it in your scripts folder.\
![img_10.png](img_10.png)

Below is a loose way of structuring your code but one that should give you some insight and intuition on how to adapt it to your game.

```csharp


```

Let's add this script as a component to an empty GameObject in the scene and then drag and drop our Steam settings, buttons, and TMP objects into their SerializedFields.
![img_17.png](img_17.png)

### Step 5 App ID:
Now that we have our Steam Transport setup, we need to get our App ID. You can get your App ID by going to the [Steamworks Developer Dashboard](https://partner.steamgames.com/dashboard/apps) or if you don't have one, you can use the "480"-app ID (Spacewar) that Steam provides. Open the folder that your project is in and at the root directory create or modify the "steam_appid.txt" file and place your ID inside and save.

![img_11.png](img_11.png)

### Step 6 Testing:
Now that we have everything set up, we can finally test our connection to Steam. Press play, and you should see a series of initialization messages in the console:
![img_15.png](img_15.png)

Press the Host button, and you should see your Host ID in the text field if the PurrNet server is running:



If you are using a friend or teammate to test, have them connect in Unity editor or over a build, and they should be able to join!

**Concluding Remarks:** Please feel free to ping me in the [PurrNet Community Discord](https://discord.gg/HnNKdkq9ta) if you found this guide helpful or have feedback on how I can improve it. This is certainly not an exhaustive guide, but I hope it helps you get started!


Cheers,

Judsin