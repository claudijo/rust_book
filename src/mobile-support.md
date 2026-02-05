# Mobile Platforms

Bevy runs on Android and iOS, bringing your games to mobile devices. While desktop and mobile share most code, platform differences require specific considerations.

## Cross-Platform Development

Write your game once and deploy to multiple platforms. Bevy abstracts platform differences where possible:

```rust
#[bevy_main]
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_system(game_logic)
        .run();
}
```

The `#[bevy_main]` attribute handles platform-specific entry points automatically. Your game logic works identically on desktop and mobile.

## Touch Input

Mobile devices use touch instead of mouse and keyboard. The touch API works across platforms:

```rust
fn handle_touches(touches: Res<Touches>) {
    for touch in touches.iter_just_pressed() {
        println!("Touch at: {:?}", touch.position());
    }
    
    for touch in touches.iter() {
        if touch.id() == 0 {
            // Primary touch - use for camera control
        }
    }
}
```

Design controls that work well with touch: large buttons, drag gestures, multi-touch support. See the Input chapter for complete touch documentation.

## Asset Loading

Mobile platforms have different file system structures:

**Android** - Assets come from the Android asset manager, not the standard file system. Bevy handles this transparently.

**iOS** - Assets bundle with the app. Access them normally through the asset server.

Both platforms use asynchronous loading. Don't assume assets are immediately available - check load state or use events.

## Performance Considerations

Mobile devices have less power than desktop:

**GPU limits** - Mobile GPUs are less capable. Reduce draw calls, use simpler shaders, and test on target devices.

**Memory constraints** - Mobile devices have limited RAM. Monitor memory usage and unload unused assets.

**Battery life** - Aggressive rendering drains batteries. Consider frame rate caps and performance modes.

**Thermal throttling** - Sustained high performance heats devices, causing CPU/GPU throttling. Optimize for sustained performance, not peak bursts.

## Screen Sizes and Orientations

Mobile screens vary widely in size and aspect ratio. Design flexible UIs:

```rust
fn responsive_ui(windows: Res<Windows>) {
    let window = windows.get_primary().unwrap();
    let width = window.width();
    let height = window.height();
    
    // Adjust UI based on screen dimensions
    let is_portrait = height > width;
    
    // Size UI elements relative to screen size
    let button_size = width * 0.2;
}
```

Handle orientation changes gracefully. Don't assume landscape or portrait - support both or lock to one orientation explicitly.

## Platform-Specific Setup

### Android

Set up Android development:
1. Install Android Studio
2. Install Android NDK
3. Add Android target: `rustup target add aarch64-linux-android`
4. Build: `cargo build --target aarch64-linux-android`

Configure your app in `AndroidManifest.xml` for permissions, orientation, and other settings.

### iOS

Set up iOS development:
1. Install Xcode
2. Add iOS target: `rustup target add aarch64-apple-ios`
3. Build: `cargo build --target aarch64-apple-ios`

Use Xcode for signing and deployment configuration.

## Testing

**Use real devices** - Emulators/simulators don't accurately represent performance. Test on actual hardware.

**Test multiple devices** - Screen sizes, GPU capabilities, and performance vary widely. Test on a range of devices.

**Monitor performance** - Profile frame times, memory usage, and battery drain. Mobile constraints are stricter than desktop.

## Best Practices

**Optimize aggressively** - Mobile devices demand efficient code. Profile and optimize more than you would for desktop.

**Handle interruptions** - Mobile apps can be interrupted by calls, notifications, or backgrounding. Save state appropriately.

**Support touch idioms** - Design for touch interaction patterns. Large targets, clear feedback, and intuitive gestures.

**Respect battery** - Provide options to reduce performance for battery savings.

**Test early and often** - Don't wait until the end to test on mobile. Platform differences can affect design decisions.

Mobile deployment expands your game's reach to billions of devices. Understanding platform constraints and designing accordingly creates experiences that work well on mobile screens with touch controls.

