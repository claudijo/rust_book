# Audio

Games need sound: music, effects, ambient audio. Bevy's audio system handles playback across platforms with a simple API.

## Playing Audio

Load and play audio files like any asset:

```rust
fn play_music(asset_server: Res<AssetServer>, audio: Res<Audio>) {
    audio.play(asset_server.load("music.ogg"));
}
```

The audio plays asynchronously. Loading happens in the background, and playback begins when the file is ready.

## Audio Sources

Bevy supports common audio formats:
- **OGG Vorbis** - Good quality, open format
- **MP3** - Widely compatible
- **WAV** - Uncompressed, high quality
- **FLAC** - Lossless compression

Different formats trade off file size, quality, and decoding cost. For music, use OGG or MP3. For short effects, WAV works well.

## Controlling Volume

Set volume when playing:

```rust
fn play_sound_effect(asset_server: Res<AssetServer>, audio: Res<Audio>) {
    audio.play_with_settings(
        asset_server.load("explosion.ogg"),
        PlaybackSettings {
            volume: 0.5,
            ..Default::default()
        },
    );
}
```

Volume ranges from 0.0 (silent) to 1.0 (full volume). Values above 1.0 amplify the audio.

## Looping

Make audio loop continuously:

```rust
fn play_background_music(asset_server: Res<AssetServer>, audio: Res<Audio>) {
    audio.play_with_settings(
        asset_server.load("theme.ogg"),
        PlaybackSettings {
            repeat: true,
            volume: 0.3,
            ..Default::default()
        },
    );
}
```

Looping audio repeats until explicitly stopped. Perfect for background music or ambient sounds.

## Spatial Audio

Position audio in 3D space:

```rust
fn play_positioned_sound(
    asset_server: Res<AssetServer>,
    audio: Res<Audio>,
    transform: &Transform
) {
    audio.play_with_settings(
        asset_server.load("footstep.ogg"),
        PlaybackSettings {
            spatial: true,
            spatial_scale: Vec3::ONE,
            ..Default::default()
        },
    );
}
```

Spatial audio creates immersive experiences by making sounds appear to come from specific locations. Volume and panning adjust based on listener position.

## Audio Sinks

For more control over playing audio, use audio sinks:

```rust
fn advanced_audio_control(
    asset_server: Res<AssetServer>,
    audio: Res<Audio>,
    mut sinks: ResMut<Assets<AudioSink>>
) {
    let handle = audio.play(asset_server.load("music.ogg"));
    
    if let Some(sink) = sinks.get_mut(&handle) {
        sink.set_volume(0.5);
        sink.pause();
        sink.play();
        sink.stop();
    }
}
```

Sinks provide runtime control: pause, resume, stop, adjust volume, and check playback state.

## Audio Events

React to audio state changes:

```rust
fn handle_audio_events(mut events: EventReader<AudioEvent>) {
    for event in events.iter() {
        match event {
            AudioEvent::Started(id) => {
                println!("Audio {} started playing", id);
            }
            AudioEvent::Finished(id) => {
                println!("Audio {} finished", id);
            }
        }
    }
}
```

Events enable synchronized behavior: starting animations when audio begins, triggering effects when sounds finish, or managing audio resource pools.

## Best Practices

**Preload audio** - Load sound effects during initialization or loading screens. Don't wait until you need to play them.

**Use appropriate formats** - OGG for music, WAV for short effects. Consider file size vs quality.

**Set reasonable volumes** - Loud audio is jarring. Start quieter than you think - players can adjust if needed.

**Limit concurrent sounds** - Too many simultaneous sounds become muddy. Pool or prioritize audio sources.

**Respect player preferences** - Provide volume controls for music and effects separately.

**Test on target platforms** - Audio behavior varies across devices. Test where your game will run.

Bevy's audio system handles the complexity of cross-platform audio while keeping playback simple. Understanding these fundamentals lets you create rich soundscapes for your games.

