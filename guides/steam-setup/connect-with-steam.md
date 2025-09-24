# Connect with Steam Using the Steam Transport
If you're looking for a quick and easy guide that will teach you how to connect with players over steam then you've come to the right place. Before we begin let's clarify what this document is and is not below:
#### This is:
✔️ A guide on how to use the Steam Transport\
✔️ A resource for how you can leverage PurrNet and connect with your friends over steam
#### This is not:
❌ A guide on how to use a lobby\
❌ A guide on how to use Steamworks.NET\
❌ A guide on how to test steam connections on a single machine

**Additional Remarks:** As a solo developer, it is often challenging to test on steam as it requires two steam accounts. There are many different ways to do this solo, but for the sake of this guide, I am going to assume that you have a friend or teammate that can help you test.

## Getting Started
This guide is being written using the following:
- Unity 6002.2.0f1
- PurrNet 1.15.0

For PurrNet installation instructions please reference the [installation guide](../../getting-started/installation-setup.md).

For reference, my Package Manager looks like the following:
![img.png](img.png)

### Step 1 Create Scene:
This is a fresh project, so I'm going to start by creating a new scene and putting it in the Assets/Scenes folder. I will call this scene "NetworkedSteamScene" and I will delete the "SampleScene" already in the folder:
![img_1.png](img_1.png)

Open the scene:

![img_2.png](img_2.png)

### Step 2 Create NetworkManager:
Right-click anywhere in the Hierarchy and go to PurrNet → NetworkManager
![img_3.png](img_3.png)

#### Step 4 Configure GameObject:
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

Select the "Unsafe" rules for now if you don't know what these are and just want to get steam working. Doing so will make the NetworkManager look like this:
![img_8.png](img_8.png)

Let's configure our NetworkManager to look like this:
![img_9.png](img_9.png)

### Step 5 Script Setup:
