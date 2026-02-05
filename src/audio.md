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

For more control over playing audio, use audio sinks. When you play audio, you receive a weak handle. Get the sink to control playback:

```rust
#[derive(Resource)]
struct BackgroundMusic(Handle<AudioSink>);

fn setup_music(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    audio: Res<Audio>,
    audio_sinks: Res<Assets<AudioSink>>
) {
    let music = asset_server.load("theme.ogg");
    let weak_handle = audio.play(music);
    
    // Upgrade to strong handle for persistent control
    let handle = audio_sinks.get_handle(weak_handle);
    commands.insert_resource(BackgroundMusic(handle));
}
```

### Controlling Playback

Control audio through the sink:

```rust
fn control_music(
    music: Res<BackgroundMusic>,
    keyboard: Res<Input<KeyCode>>,
    audio_sinks: Res<Assets<AudioSink>>
) {
    if let Some(sink) = audio_sinks.get(&music.0) {
        if keyboard.just_pressed(KeyCode::P) {
            sink.pause();
        }
        
        if keyboard.just_pressed(KeyCode::R) {
            sink.play();  // Resume playback
        }
    }
}
```

The release notes show `.pause()` and `.play()` methods for controlling playback.

### Volume Control

Adjust volume dynamically:

```rust
fn adjust_volume(
    music: Res<BackgroundMusic>,
    keyboard: Res<Input<KeyCode>>,
    mut audio_sinks: ResMut<Assets<AudioSink>>
) {
    if let Some(sink) = audio_sinks.get_mut(&music.0) {
        if keyboard.pressed(KeyCode::Up) {
            sink.set_volume(sink.volume() + 0.01);
        }
        if keyboard.pressed(KeyCode::Down) {
            sink.set_volume((sink.volume() - 0.01).max(0.0));
        }
    }
}
```

### Speed Control

Change playback speed for effects:

```rust
fn change_speed(
    audio_sinks: Res<Assets<AudioSink>>,
    handle: &Handle<AudioSink>
) {
    if let Some(sink) = audio_sinks.get(handle) {
        sink.set_speed(2.0);   // Double speed
        sink.set_speed(0.5);   // Half speed
        sink.set_speed(1.0);   // Normal speed
    }
}
```

Speed control enables slow-motion effects, fast-forward narration, or pitch-shifted sound effects.

The release notes confirm these AudioSink control methods: `.pause()`, `.play()`, `.set_volume()`, `.volume()`, and `.set_speed()`. Consult the Bevy API documentation for additional AudioSink methods.

## Best Practices

**Preload audio** - Load sound effects during initialization or loading screens. Don't wait until you need to play them.

**Use appropriate formats** - OGG for music, WAV for short effects. Consider file size vs quality.

**Set reasonable volumes** - Loud audio is jarring. Start quieter than you think - players can adjust if needed.

**Limit concurrent sounds** - Too many simultaneous sounds become muddy. Pool or prioritize audio sources.

**Respect player preferences** - Provide volume controls for music and effects separately.

**Test on target platforms** - Audio behavior varies across devices. Test where your game will run.

Bevy's audio system handles the complexity of cross-platform audio while keeping playback simple. Understanding these fundamentals lets you create rich soundscapes for your games.

