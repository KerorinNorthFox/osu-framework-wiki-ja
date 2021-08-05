# Tracks

Tracks are for playing single-instanced long-form audio. They offer the widest variety of adjustments possible, but are expensive and not recommended to be used for quick-fire or concurrent audio playback.

- **`ITrackStore`**: Used to retrieve `Track` objects. The global track store is accessible through either `AudioManager.Tracks` or by resolving an `ITrackStore` dependency into the `Drawable` object.
- **`Track`**: The track, which is usually stored at a somewhat global level and played as required.

The global track store provides access to all `.mp3` files placed in the application's `Resources/Tracks/` directory.

The following is a typical usage pattern:

```csharp
private Track track;
private DrawableTrack drawableTrack;

[BackgroundDependencyLoader]
private void load(ITrackStore tracks)
{
    // Option 1: Raw tracks.
    // This only has the audio adjustments (e.g. volume) of the track store applied to it.
    track = tracks.Get("test-track.mp3");

    // Option 2: Drawable tracks.
    // Along with the track store's own adjustments, this version also has the adjustments of any parenting AudioContainers applied to it.
    drawableTrack = new DrawableTrack(tracks.Get("test-sample.mp3"));
    Child = drawableTrack;

    // Typically, either of the above are stored in a field in the Drawable object to be used later on. For example, in OnClick().

    // You can adjust the volume of all tracks retrieved from the track store. Note that this is a global property.
    // E.g. The track playback volume is calculated as: (track_store_volume) * (track_volume).
    tracks.Volume.Value = 0.8;

    // You can also adjust other properties on the track itself.
    track.Volume.Value = 0.8;
}

private bool playing;

protected override bool OnClick(ClickEvent e)
{
    if (!playing)
        track.Start();
    else
        track.Stop();

    playing = !playing;
    return true;
}

protected override bool OnDoubleClick(DoubleClickEvent e)
{
    track.Seek(0);
    return true;
}

protected override void Dispose(bool isDisposing)
{
    base.Dispose(isDisposing);

    // Be sure to dispose the track, otherwise memory will be leaked!
    // This is automatic for DrawableTrack.
    track.Dispose();
}
```

## Virtual tracks

`ITrackStore.GetVirtual()` can be used as a sane fallback value for cases where a non-null `Track` is always required. It mimics all positional methods of a non-virtual track without playing audio.

## Custom track stores

In order to retrieve tracks from a different location than the default `Resources/Tracks/` directory, a custom track store is required.

To create a custom track store, pass a `ResourceStore` to `AudioManager.GetTrackStore()`. This will return an `ITrackStore` to be used for future lookups. The custom track store will have its audio (e.g. volume) adjusted by the global sample store

```csharp
[BackgroundDependencyLoader]
private void load(AudioManager audio)
{
    var trackStore = audio.GetTrackStore(new ResourceStore<byte[]>(...));
    trackStore.Get("test-track.mp3");
}
```

# Samples

Samples are for fast and concurrent playback of short audio clips. They feature automatic memory management with the intention of being fired-and-forgotten, but lack some notable features of `Track` such as time tracking, seeking, tempo adjustments, and on-demand restarting of playback.

- **`ISampleStore`**: Used to retrieve `Sample` objects. The global sample store is accessible through either `AudioManager.Samples` or by resolving an `ISampleStore` dependency into the `Drawable` object.
- **`Sample`**: The sample, which is typically stored by the `Drawable` object in some fashion - either as the raw `Sample` or nested inside a `DrawableSample` object. Used in order to retrieve and play one or more `SampleChannel`s.
- **`SampleChannel`**: The object which plays audio. This is typically fired-and-forgotten, with special cases in-case looping, adjusting volume/balance/frequency, or stopping audio playback is required.

The global sample store provides access to all `.wav` and `.mp3` files placed in the application's `Resources/Samples/` directory.

The following is a typical usage pattern:
```cs
private Sample sample;
private DrawableSample drawableSample;

[BackgroundDependencyLoader]
private void load(ISampleStore samples)
{
    // Option 1: Raw samples.
    // This only has the audio adjustments (e.g. volume) of the sample store applied to it.
    sample = samples.Get("test-sample.mp3");

    // Option 2: Drawable samples.
    // Along with the sample store's own adjustments, this version also has the adjustments of any parenting AudioContainers applied to it.
    drawableSample = new DrawableSample(samples.Get("test-sample.mp3"));
    Child = drawableSample;

    // Typically, either of the above are stored in a field in the Drawable object to be used later on. For example, in OnClick().

    // You can adjust the volume of all samples retrieved from the sample store. Note that this is a global property.
    // E.g. The channel playback volume is calculated as: (sample_store_volume) * (sample_volume) * (channel_volume).
    samples.Volume.Value = 0.8;
}

protected override bool OnClick(ClickEvent e)
{
    // The following examples demonstrate usage of the raw Sample object, but DrawableSample can be used interchangeably.

    // Play a normal channel from the sample.
    // This can be called as many times as desired, but concurrent playback will only be heard
    // up to sample.PlaybackConcurrency times from the same sample name (e.g. "test-sample.mp3").
    sample.Play();

    // Play another normal channel. We'll be using this one for more examples later on.
    var channel2 = sample.Play();

    // Play a looping channel.
    // It's recommended to retrieve the channel to set the looping flag first, and then to play the channel afterwards.
    // The same applies to adjusting other channel properties, e.g. setting an initial volume.
    var channel1 = sample.GetChannel();
    channel1.Looping = true;
    channel1.Play();

    // Apply a volume adjustment to all channels (all three above, and any future ones) from the sample.
    sample.Volume.Value = 0.5;

    // Apply an independent volume adjustment to channel 1 only.
    // The final volume of this channel will be 0.8 (store) * 0.5 (sample) * 0.5 (channel) = 0.2.
    channel1.Volume.Value = 0.5;

    // Stop channel 2, but keep channel 1 playing.
    channel2.Stop();

    // Resume playback of channel 2. This doesn't restart unless playback has finished.
    channel2.Play();

    return true;
}

protected override void Dispose(bool isDisposing)
{
    base.Dispose(isDisposing);

    // Halts playback of all played channels from the sample. This is automatic for DrawableSample.
    // Note: Omission doesn't always result in a memory leak, however it's recommended to stop
    // longer sample playback when Drawables are disposed, and especially so for looping channels.
    sample.Dispose();
}
```

## Virtual samples

`SampleVirtual` can be used as a sane fallback value for cases where a non-null `Sample` is always required.

## Custom sample stores

In order to retrieve samples from a different location than the default `Resources/Samples/` directory, a custom sample store is required.

To create a custom sample store, pass a `ResourceStore` to `AudioManager.GetSampleStore()`. This will return an `ISampleStore` to be used for future lookups. The custom sample store will have its audio (e.g. volume) adjusted by the global sample store

```csharp
[BackgroundDependencyLoader]
private void load(AudioManager audio)
{
    var sampleStore = audio.GetSampleStore(new ResourceStore<byte[]>(...));
    sampleStore.Get("test-sample.mp3");
}
```

# Mixing

All `SampleChannel` and `Track` audio (hereby referred to as a "channel") is routed through an `AudioMixer`. DSP effects can be applied to the `AudioMixer` to change the resultant audio of channels routed through it independent of other `AudioMixers` in the game.

## Global audio mixer

The global audio mixer affects all channels by default and can be accessed through `AudioManager.Mixer`. The following is a simple example of how to apply a DSP effect globally:
```csharp
[BackgroundDependencyLoader]
private void load(AudioManager audio, ISampleStore samples)
{
    // Add an effect to the global mixer.
    audio.Mixer.Effects.Add(new ...);

    // The sample has the above effect applied to it by default.
    samples.Get(...).Play();

    // The same is true for DrawableSample/DrawableTrack.
    DrawableSample drawableSample;
    Add(drawableSample = new DrawableSample(samples.Get(...)));
    drawableSample.Play();
}
```

## Local (custom) audio mixers

The global mixer provides too wide of an effect for use in many cases. There are two ways to create a mixer for only the specific audio which needs to be affected.

One option is to create a locally managed `AudioMixer` through `AudioManager.CreateAudioMixer()`:
```csharp
private AudioMixer mixer;

[BackgroundDependencyLoader]
private void load(AudioManager audio, ISampleStore samples)
{
    // Create the custom mixer.
    mixer = audio.CreateAudioMixer();

    // Add an effect to the mixer.
    mixer.Effects.Add(new ...);

    Sample sample = samples.Get(...);

    // Add channels to the custom mixer in order to apply the effect to them.
    SampleChannel channel = sample.GetChannel();
    mixer.Add(channel);
    channel.Play();

    // Channels that are not added to the custom mixer will only be affected by the global mixer.
    sample.Play();
}

protected override void Dispose(bool isDisposing)
{
    base.Dispose(isDisposing);

    // Be sure to dispose the mixer, otherwise memory will be leaked!
    mixer?.Dispose();
}
```

The other option is to use `DrawableAudioMixer`, which takes care of the hard work behind the scenes:
```csharp
[BackgroundDependencyLoader]
private void load(ISampleStore samples)
{
    DrawableAudioMixer drawableAudioMixer;
    DrawableSample drawableSample;

    // Add a mixer to the hierarchy, this will affect DrawableSample/DrawableTrack children added to it, recursing until another DrawableAudioMixer is reached.
    // Note: It will not affect Sample/SampleChannel/Tracks unless they're added manually!
    Add(drawableAudioMixer = new DrawableAudioMixer
    {
        Child = drawableSample = new DrawableSample(samples.Get(...)),
        Effects =
        {
            ...
        }
    });

    // This sample has the above effect applied to it.
    drawableSample.Play();

    // As above, you can still add channels to the mixer manually to apply the effect to them.
    Sample sample = samples.Get(...);
    SampleChannel channel = sample.GetChannel();
    drawableAudioMixer.Add(channel);
    channel.Play();
}
```

## Effect prioritisation

Effects are applied in the order that they appear in the `AudioMixer.Effects` or `DrawableAudioMixer.Effects` list. Effects can be added, removed, or moved around in order to change their priority.

## Changing the audio mixer

Channels can be moved to another audio mixer by adding or removing them from the `AudioMixer`:
```csharp
[BackgroundDependencyLoader]
private void load(ISampleStore samples)
{
    DrawableAudioMixer mixer1;
    DrawableAudioMixer mixer2;

    Children = new Drawable[]
    {
        mixer1 = new DrawableAudioMixer(),
        mixer2 = new DrawableAudioMixer()
    };

    // Add the channel to the first mixer.
    SampleChannel channel;
    mixer1.Add(channel = samples.Get(...).Play());

    // Move the channel to the second mixer.
    mixer2.Add(channel);

    // Remove the channel from the second mixer.
    // Note: The channel will be moved to the global mixer, not mixer1 from above!
    mixer2.Remove(channel);

    // This has no effect. All channels must be played by one mixer.
    audio.Mixer.Remove(channel);



    // Similarly, for DrawableSample:
    DrawableSample drawableSample = new DrawableSample(samples.Get(.));
    SampleChannel channel1 = drawableSample.Play();
    SampleChannel channel2 = drawableSample.Play();

    // Add the entire sample to the first mixer, this affects both played channels.
    mixer1.Add(drawableSample);

    // Move the entire sample to the second mixer, this affects both played channels.
    mixer1.Remove(drawableSample);
    mixer2.Add(drawableSample);

    // Move the first channel to the first mixer.
    mixer1.Add(channel1);

    // Move the second channel to the global mixer.
    mixer2.Remove(channel2);

    // Move the entire sample to the first mixer, this still affects both played channels.
    mixer2.Remove(drawableSample);
    mixer1.Add(drawableSample);
}
```