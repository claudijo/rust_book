# Windows

Bevy provides a backend-agnostic windowing API.

## Creating Windows

Windows are created automatically when using the default plugins:

```rust
fn main() {
    App::build()
        .add_plugins(DefaultPlugins)
        .run();
}
```

## Window Configuration

You can configure the initial window settings:

```rust
fn main() {
    App::build()
        .add_plugins(DefaultPlugins)
        .add_resource(WindowDescriptor {
            title: "My Game".to_string(),
            width: 800.0,
            height: 600.0,
            vsync: true,
            resizable: true,
            ..Default::default()
        })
        .run();
}
```

## Dynamic Window Settings

**Added in Bevy 0.3**

Prior to 0.3, window settings could only be set once at app startup. Bevy 0.3 added the ability to dynamically modify window properties at runtime:

```rust
fn change_title(time: Res<Time>, mut windows: ResMut<Windows>) {
    let window = windows.get_primary_mut().unwrap();
    window.set_title(format!(
        "Seconds since startup: {}", 
        time.seconds_since_startup()
    ));
}
```

### Available Dynamic Settings

You can change these properties at runtime:
- **Title**: `window.set_title()`
- **Resolution**: `window.set_resolution()`
- **Vsync**: `window.set_vsync()`
- **Resizable**: `window.set_resizable()`
- **Decorations**: `window.set_decorations()`
- **Cursor visibility**: `window.set_cursor_visibility()`
- **Cursor lock**: `window.set_cursor_lock_mode()`
- **Position**: `window.set_position()`
- **Maximized**: `window.set_maximized()`
- **Minimized**: `window.set_minimized()`
- **Mode**: `window.set_mode()` (windowed, fullscreen, etc.)

### Multiple Windows

```rust
fn system(mut windows: ResMut<Windows>) {
    // Access the primary window
    if let Some(window) = windows.get_primary_mut() {
        window.set_title("Primary Window");
    }
    
    // Access by ID
    if let Some(window) = windows.get_mut(window_id) {
        window.set_title("Secondary Window");
    }
}
```

## Window Events

React to window events:

```rust
fn window_events(mut events: EventReader<WindowEvent>) {
    for event in events.iter() {
        match event {
            WindowEvent::Resized { width, height, .. } => {
                println!("Window resized to {}x{}", width, height);
            }
            WindowEvent::Focused(focused) => {
                println!("Window focus: {}", focused);
            }
            _ => {}
        }
    }
}
```

## Platform-Specific Features

### Mouse Capture

**Added in Bevy 0.3**

```rust
fn capture_mouse(mut windows: ResMut<Windows>) {
    let window = windows.get_primary_mut().unwrap();
    window.set_cursor_lock_mode(CursorLockMode::Locked);
    window.set_cursor_visibility(false);
}
```

### Window Decorations

Control window decorations (title bar, borders):

```rust
fn toggle_decorations(mut windows: ResMut<Windows>) {
    let window = windows.get_primary_mut().unwrap();
    window.set_decorations(!window.decorations());
}
```

### Window Transparency

**Added in Bevy 0.6**

Create transparent windows for widget-like applications:

```rust
fn main() {
    App::new()
        .insert_resource(WindowDescriptor {
            transparent: true,
            decorations: false,
            ..Default::default()
        })
        .add_plugins(DefaultPlugins)
        .run();
}
```

This allows you to create apps that blend with the desktop background, perfect for:
- Desktop widgets
- Overlay applications  
- Transparent HUDs
- Borderless applications

**Note:** Platform support varies. Works well on Windows, macOS, and Linux (with compositing).

### WASM Canvas

On WASM, you can provide an existing canvas element:

```rust
// Platform-specific configuration for WASM
#[cfg(target_arch = "wasm32")]
{
    use bevy::winit::WinitSettings;
    app.add_resource(WinitSettings {
        canvas: Some("#my-canvas".to_string()),
        ..Default::default()
    });
}
```

## Best Practices

### Responsive Window Changes

When changing window settings based on game state, use change detection to avoid unnecessary updates:

```rust
fn update_window(
    game_state: Res<GameState>,
    mut windows: ResMut<Windows>
) {
    if game_state.is_changed() {
        let window = windows.get_primary_mut().unwrap();
        match *game_state {
            GameState::Menu => {
                window.set_cursor_visibility(true);
            }
            GameState::Playing => {
                window.set_cursor_visibility(false);
            }
        }
    }
}
```

### Handle Window Close

```rust
fn exit_on_close(mut app_exit: EventWriter<AppExit>, windows: Res<Windows>) {
    if windows.get_primary().map(|w| w.should_close()).unwrap_or(false) {
        app_exit.send(AppExit);
    }
}
```

Dynamic window settings make it easy to respond to user preferences, game state changes, and create polished user experiences.

