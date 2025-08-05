# Lip Sync

Adding lip syncing to your multiplayer game is now easier than ever. We have some pre-built components to make your developer experience easy as ever.

### Sprite

Simply add the **PurrLipSyncSprite** component to your player prefab and give it a reference to the **PurrVoicePlayer** component. From here it needs a **PhonemeSpritePreset**, which essentially holds the different sprites for various mouth shapes (A, E, O, U, etc.)

Lastly you feed it your sprite renderer and the rest should be handled from there!

<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

### Other

Simply add the **PurrLipSync** component to your player prefab and give it a reference to the **PurrVoicePlayer** component.

There is a public value called **result** which holds LipSyncInfo. You can utilize this info to handle the various lip syncing setups.

#### LipSyncInfo:

* phoneme
* volume
* rawVolume
* phonemeRatios
