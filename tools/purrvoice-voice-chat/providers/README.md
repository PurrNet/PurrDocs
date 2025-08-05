# Providers

The provider pattern is utilized for this voice chat setup, in order to make it very easy to use and highly customizable to use. It comes with some default providers, but it should also be pretty straight forward making your own!

The providers are required for both input and output. If you just wish to use the Unity audio system, you should likely go with [Default Input Provider](input-providers/default-input-provider.md) and [Audio Source Voice Provider](output-providers/audio-source-voice-provider.md). These utilize the default Unity audio source setup and Unity input handling for devices as well.

Keep in mind that you need 2 individual output providers if you want local playback (typically used for testing). This is because the audio is handled on an instance basis, meaning that the handler for local and non local playback shouldn't be mixed! The PurrVoicePlayer should warn you in case of this mistake either way, but it's good to know why.
