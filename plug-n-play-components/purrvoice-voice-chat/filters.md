# Filters

Working with the filters should be very straight forward. They allow easy audio manipulation to get the voice chat effect you desire for your game!

<figure><img src="../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

### Filter level

Filters allow you to easily manipulate audio at any level of it's path. This in turn means that you can easily control whether the Sender, Server or Receiver will be the one applying the filter to the audio.

**Sender:** This means that the audio is applied before it even reached the network, and as soon as input is received from your microphone device. This level is optimal for things such as noise cancellation, or funny effects that should be applied globally.

**Server:** The server will de-compress the audio, apply the filter and compress it before sending to other clients. Generally this layer shouldn't be utilized heavily, as it can get computationally heavy for the server/host to handle all audio manipulation. But if you care about cheat proofing your voice, this is a good place to do so.

**Receiver:** Here the receiver will be the one applying the effect. This is great for environment context relevant effects, such as a player being behind the wall for you, or under the water, etc. Essentially your local client sees someone else in some relevant context and need a filter reacting to that.

### Filter Strength

The strength of a filter is handled on a per filter level, so what it "means" can vary depending on the intention of a filter, so it can be a good idea to play around with this value to see the effect of this. The strength would generally mean how much a filters effect is applied to the voice, but again, this can vary.

The strength of a filter is NOT dynamically synced, as it is rather pointless. Only for the level manipulating it, is it relevant to know a dynamic strength, so in order to save data we are not dynamically syncing this.

Changing the strength through code is simple:

```csharp
[SerializeField] private PurrVoicePlayer _voicePlayer;
[SerializeField] private PurrAudioFilter _testFilter;

private void SetStrength(float strength)
{
    // Filter reference/Index, new strength
    _voicePlayer.SetFilterStrength(_testFilter, strength);
}
```

### Dynamically adding filters

You can add and remove filters on the fly, and this will be synchronized initially. It's a very lightweight action to do, and only at this level is the strength synced (initial value)

This can only be done by the controller.

Here is a simple example:

```csharp
[SerializeField] private PurrVoicePlayer _voicePlayer;
[SerializeField] private PurrAudioFilter _testFilter;

private void ApplyFilter()
{
    // Filter, Level, Initial strength
    _voicePlayer.AddFilter(_testFilter, FilterLevel.Receiver, 1);
}

private void RemoveFilter()
{
    // This can be both a direct filter reference or an index of filter
    _voicePlayer.RemoveFilter(_testFilter);
}
```
