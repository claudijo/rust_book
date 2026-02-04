# Mobile Platform Support

**Added in Bevy 0.3**

Bevy 0.3 introduced initial support for Android and iOS platforms.

## Android Support

Initial Android support was added through a massive group effort:

### What Works

- Bevy ECS schedules
- Touch input
- Asset loading via Android Asset Manager
- Audio playback (via cpal)
- Basic rendering

### Setup

You can try the Bevy Android example by following the instructions in the repository. 

### Implementation Details

The Android support involved work across multiple areas:
- Rewrote bevy-glsl-to-spirv to support Android/static libraries
- Asset backend using Android Asset Manager
- Touch input support
- Texture format compatibility
- UI touch support including touch force

### Current Status

This is very fresh - many features work but some don't. This is a great time to help close the gaps!

## iOS Support

Bevy can now run on iOS!

### What Works

- Bevy ECS schedules
- Touch input
- Basic rendering
- Window management

### Setup

You can try the Bevy iOS example by following the instructions in the repository.

### Implementation Details

iOS support involved:
- XCode project setup and examples
- Runtime shader compilation using shaderc
- Rodio upgrade for iOS audio
- Touch input support
- Winit fixes for iOS portrait view
- RustAudio iOS support

### Known Issues

- Audio doesn't quite work yet

## Touch Input

Both platforms use the unified touch input API:

```rust
fn touch_system(touches: Res<Touches>) {
    for touch in touches.iter() {
        println!("Touch at: {:?}", touch.position());
    }
    
    for touch in touches.iter_just_pressed() {
        println!("New touch: {:?}", touch);
    }
}
```

See the [Input chapter](./input.md) for complete touch input documentation.

## Cross-Platform Considerations

### Asset Loading

Each platform uses appropriate storage:
- **Android**: Android Asset Manager via AssetIo
- **iOS**: Standard filesystem
- Both use async loading

### Rendering

Mobile platforms have specific requirements:
- Shader compatibility (GLES vs Metal)
- Texture format support
- Performance considerations

### Input

Touch input is the primary input method on mobile. See [Touch Input](./input.md#touch-input) for details.

## Future Work

Both Android and iOS support are in early stages. Expected improvements:
- Better audio support (especially iOS)
- Performance optimizations
- More complete feature coverage
- Better platform integration
- Additional mobile-specific features

The foundation is solid and ready for community contributions to close the remaining gaps!

