# Microphone Input

You can easily see all microphone devices and change device as you want, allowing for an easy setup of settings and good runtime flexibility.

You can also easily see your current device in the PurrVoicePlayer:

<figure><img src="../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

### Getting microphone devices

Getting the devices is easy, as we've already setup the class you need for this. You can easily call the AudioDevices from any script, which holds a "read only" list of all devices that you can easily access like so:

```csharp
private IReadOnlyList<Device> GetDevices() {
    return AudioDevices.devices;
}
```

### Changing microphone

Changing microphone is just a single call to your local PurrVoicePlayer. \
This simple example below will set your microphone to the first device in the devices list.&#x20;

```csharp
[SerializeField] private PurrVoicePlayer _voicePlayer; // This should be our local player

private void MicChangeTest()
{
    _voicePlayer.ChangeMicrophone(AudioDevices.devices[0]);
}
```
