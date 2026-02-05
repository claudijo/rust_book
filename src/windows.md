# Windows

Bevy's window system abstracts platform differences, letting you create and manage windows with a consistent API across desktop, web, and mobile.

## Window Creation

Windows are created automatically when using the default plugins:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .run();
}
```

This creates a single window with default settings: 1280x720 resolution, a generic title, and standard decorations.

## Initial Configuration

Customize the window before it opens using WindowDescriptor:

```rust
fn main() {
    App::new()
        .insert_resource(WindowDescriptor {
            title: "My Game".to_string(),
            width: 1920.0,
            height: 1080.0,
            resizable: true,
            ..Default::default()
        })
        .add_plugins(DefaultPlugins)
        .run();
}
```

This sets properties before window creation. Some properties can be changed later, others are fixed at creation time.

## Accessing Windows

Query the Windows resource to access window state:

```rust
fn inspect_window(windows: Res<Windows>) {
    if let Some(window) = windows.get_primary() {
        println!("Window size: {}x{}", window.width(), window.height());
        println!("Window is focused: {}", window.is_focused());
    }
}
```

Most games work with a single primary window. Multi-window support exists for editor tools or complex applications.

## Runtime Changes

Modify window properties while your game runs:

```rust
fn toggle_fullscreen(
    keyboard: Res<Input<KeyCode>>,
    mut windows: ResMut<Windows>
) {
    if keyboard.just_pressed(KeyCode::F11) {
        let window = windows.get_primary_mut().unwrap();
        
        match window.mode() {
            WindowMode::Windowed => {
                window.set_mode(WindowMode::BorderlessFullscreen);
            }
            _ => {
                window.set_mode(WindowMode::Windowed);
            }
        }
    }
}
```

## Window Properties

### Title

Update the title bar text:

```rust
fn update_title(
    score: Res<Score>,
    mut windows: ResMut<Windows>
) {
    if score.is_changed() {
        let window = windows.get_primary_mut().unwrap();
        window.set_title(format!("My Game - Score: {}", score.value));
    }
}
```

### Resolution

Resize the window programmatically:

```rust
window.set_resolution(1920.0, 1080.0);
```

Users can also resize windows by dragging edges (if `resizable` is true).

### Display Mode

Switch between windowed and fullscreen modes:

```rust
// Different fullscreen modes
window.set_mode(WindowMode::Windowed);
window.set_mode(WindowMode::BorderlessFullscreen);  // Fullscreen without mode change
window.set_mode(WindowMode::SizedFullscreen);       // Exclusive fullscreen
window.set_mode(WindowMode::Fullscreen);            // Native fullscreen
```

Borderless fullscreen is fastest to switch and works well for most games. Exclusive fullscreen may have lower latency but takes longer to alt-tab.

### VSync

Control vertical synchronization:

```rust
window.set_vsync(true);   // Cap FPS to monitor refresh rate
window.set_vsync(false);  // Uncapped FPS
```

VSync prevents screen tearing but may add input latency. Competitive games often disable it.

### Decorations

Hide or show window borders and title bar:

```rust
window.set_decorations(false);  // Borderless window
```

Useful for custom window chrome or immersive experiences.

## Cursor Control

### Visibility

Show or hide the cursor:

```rust
fn hide_cursor_in_game(
    state: Res<State<GameState>>,
    mut windows: ResMut<Windows>
) {
    let window = windows.get_primary_mut().unwrap();
    
    match state.current() {
        GameState::Playing => window.set_cursor_visibility(false),
        GameState::Menu => window.set_cursor_visibility(true),
    }
}
```

### Cursor Lock

Lock the cursor to the window for first-person controls:

```rust
fn lock_cursor(mut windows: ResMut<Windows>) {
    let window = windows.get_primary_mut().unwrap();
    window.set_cursor_lock_mode(true);
    window.set_cursor_visibility(false);
}
```

When locked, the cursor stays within the window and mouse movement generates relative motion events perfect for camera controls.

### Cursor Position

Set the cursor position in window coordinates:

```rust
fn center_cursor(mut windows: ResMut<Windows>) {
    let window = windows.get_primary_mut().unwrap();
    let center_x = window.width() / 2.0;
    let center_y = window.height() / 2.0;
    window.set_cursor_position(Vec2::new(center_x, center_y));
}
```

## Window Events

React to window state changes:

```rust
fn handle_window_events(mut events: EventReader<WindowResized>) {
    for event in events.iter() {
        println!("Window resized to {}x{}", event.width, event.height);
        // Update camera, UI, etc.
    }
}
```

Common window events include:
- `WindowResized` - Window size changed
- `WindowFocused` - Window gained or lost focus
- `WindowMoved` - Window position changed
- `WindowCloseRequested` - User clicked the close button

### Handling Close Requests

By default, clicking the close button exits the app. Intercept it to show a confirmation:

```rust
fn handle_close(
    mut events: EventReader<WindowCloseRequested>,
    mut app_exit: EventWriter<AppExit>
) {
    for _event in events.iter() {
        // Show confirmation dialog
        if user_confirmed_exit() {
            app_exit.send(AppExit);
        }
    }
}
```

## Window Transparency

Create windows with transparent backgrounds:

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

The window becomes see-through where nothing renders. This enables overlay applications, desktop widgets, or creative window shapes. Platform support varies - works well on Windows, macOS, and Linux with compositing.

## Multiple Windows

While most games use one window, you can create multiple:

```rust
fn spawn_secondary_window(mut commands: Commands, mut windows: ResMut<Windows>) {
    let window_id = windows.add(WindowDescriptor {
        title: "Tool Window".to_string(),
        width: 400.0,
        height: 300.0,
        ..Default::default()
    });
    
    // Store the ID for later access
    commands.insert_resource(ToolWindowId(window_id));
}
```

Access specific windows by ID:

```rust
fn update_tool_window(
    tool_id: Res<ToolWindowId>,
    mut windows: ResMut<Windows>
) {
    if let Some(window) = windows.get_mut(tool_id.0) {
        window.set_title("Updated Title");
    }
}
```

## Platform Considerations

### Desktop

Desktop windows support all features: resizing, fullscreen, transparency, multiple windows.

### Web

Web builds run in the browser. Configure the canvas element:

```rust
#[cfg(target_arch = "wasm32")]
fn configure_web_window(mut windows: ResMut<Windows>) {
    let window = windows.get_primary_mut().unwrap();
    window.set_canvas("#game-canvas");
}
```

Some features like fullscreen require browser APIs and user interaction.

### Mobile

Mobile platforms use fullscreen by default. Window resize events reflect orientation changes and system UI visibility.

## Best Practices

**Use change detection** - Only update window properties when necessary:

```rust
fn update_window(
    settings: Res<GraphicsSettings>,
    mut windows: ResMut<Windows>
) {
    if settings.is_changed() {
        let window = windows.get_primary_mut().unwrap();
        window.set_vsync(settings.vsync);
    }
}
```

**Handle resize events** - Update cameras and UI when the window size changes:

```rust
fn resize_camera(
    mut events: EventReader<WindowResized>,
    mut camera: Query<&mut OrthographicProjection>
) {
    for event in events.iter() {
        for mut projection in camera.iter_mut() {
            projection.update(event.width, event.height);
        }
    }
}
```

**Save window state** - Remember fullscreen preferences and window position:

```rust
fn save_window_state(windows: Res<Windows>, mut settings: ResMut<GameSettings>) {
    if let Some(window) = windows.get_primary() {
        settings.fullscreen = matches!(window.mode(), WindowMode::Fullscreen);
        settings.resolution = (window.width(), window.height());
    }
}
```

**Provide window controls** - Let players configure window settings through menus rather than forcing a single mode.

Bevy's window system handles platform complexity while providing fine-grained control. Understanding window management helps create polished, flexible applications that work well across all platforms.

