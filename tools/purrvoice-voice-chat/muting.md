# Muting

Handling muting using the PurrVoicePlayer is simple as ever. First of all, there is an exposed "muted" bool in the inspector, toggling this will reveal it's effects.

**Locally Muted** - If you mute your own player, no audio data will be sent over the network, and no processing will happen.

**Remove muted** - If you mute a remote player (another client) you'll locally not playback their audio.

You can easily handle this through script like so:\


```csharp
[SerializeField] private PurrVoicePlayer _voicePlayer;

private void SetMute(bool mute)
{
    _voicePlayer.muted = mute;
}
```
