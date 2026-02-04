# Sound

Bevy provides audio playback capabilities. The audio system will be extended with more control and features in the future.

## Loading and Playing Sounds

You can load and play sounds like this:

```rust
fn system(asset_server: Res<AssetServer>, audio_output: Res<AudioOutput>) {
    let music: Handle<AudioSource> = asset_server.load("music.mp3").unwrap();

    // this will play the music asynchronously as soon as it has loaded
    audio_output.play(music);

    // if you already have an AudioSource reference you can play it immediately like this:
    audio_output.play_source(audio_source);
}
```

## Future Plans

We plan on extending the audio system with more control and features in the future.

## Bevy 0.2 Improvements

### Custom Rodio Source

**Added in Bevy 0.2**

Custom rodio source for audio support was added. `AudioOutput` is now able to play anything that implements the `Decodable` trait, providing much more flexibility in audio formats and sources.

