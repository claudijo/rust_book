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

