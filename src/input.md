# Input

Bevy provides cross-platform input handling for keyboard, mouse, and gamepads.

## Keyboard Input

The `Input<KeyCode>` resource tracks keyboard state:

```rust
fn system(keys: Res<Input<KeyCode>>) {
    if keys.pressed(KeyCode::Space) {
        println!("Space is held down");
    }
    if keys.just_pressed(KeyCode::Return) {
        println!("Enter was just pressed this frame");
    }
}
```

## Mouse Input

Mouse button state and events:

```rust
fn system(mouse: Res<Input<MouseButton>>) {
    if mouse.just_pressed(MouseButton::Left) {
        println!("Left mouse button clicked");
    }
}
```

## Gamepad Input

**Added in Bevy 0.2**

The Bevy Input plugin now has cross-platform support for most controllers thanks to the gilrs library!

```rust
fn button_system(
    gamepads: Res<Vec<Gamepad>>, 
    button_input: Res<Input<GamepadButton>>
) {
    for gamepad in gamepads.iter() {
        if button_input.just_pressed(GamepadButton(*gamepad, GamepadButtonType::RightTrigger)) {
            println!("Pressed right trigger!");
        }
    }
}
```

### Gamepad Detection

Connected gamepads are available in the `Vec<Gamepad>` resource. You can iterate over them to handle input from multiple controllers.

### Button Types

Gamepad buttons include:
- Face buttons (A, B, X, Y / Cross, Circle, Square, Triangle)
- D-pad directions
- Shoulder buttons and triggers
- Start, Select, and other standard buttons

### Gamepad Settings

**Added in Bevy 0.3**

The `GamepadSettings` resource allows customization of gamepad behavior on a per-controller, per-axis/button basis:

```rust
fn setup_gamepad(mut gamepad_settings: ResMut<GamepadSettings>) {
    gamepad_settings.axis_settings.insert(
        GamepadAxis(Gamepad(0), GamepadAxisType::LeftStickX),
        AxisSettings {
            positive_high: 0.8,     // Upper deadzone
            positive_low: 0.01,     // Lower deadzone
            ..Default::default()
        },
    );
}
```

This allows you to:
- Set deadzones for analog sticks
- Configure trigger sensitivity
- Customize per-controller settings
- Fine-tune input response

## Input Iterator Methods

**Added in Bevy 0.2**

New methods on `Input<T>` provide iterator access to input state:

```rust
fn system(keys: Res<Input<KeyCode>>) {
    // Iterate over all currently pressed keys
    for key in keys.get_pressed() {
        println!("Key pressed: {:?}", key);
    }
    
    // Iterate over keys just pressed this frame
    for key in keys.get_just_pressed() {
        println!("Key just pressed: {:?}", key);
    }
    
    // Iterate over keys just released this frame
    for key in keys.get_just_released() {
        println!("Key just released: {:?}", key);
    }
}
```

## Input State

The input system automatically tracks three states for each input:
- **Pressed**: Currently being held down
- **Just Pressed**: Was pressed this frame
- **Just Released**: Was released this frame

This makes it easy to handle both continuous input (like movement) and discrete input (like menu navigation).

## Touch Input

**Added in Bevy 0.3**

Bevy supports touch input for mobile and touch-enabled devices:

```rust
fn touch_system(touches: Res<Touches>) {
    // Iterate all current touches and retrieve their state
    for touch in touches.iter() {
        println!("active touch: {:?}", touch);
        println!("  position: {}", touch.position());
        println!("  force: {:?}", touch.force());
    }

    // Just pressed this frame
    for touch in touches.iter_just_pressed() {
        println!("just pressed {:?}", touch);
    }

    // Just released this frame
    for touch in touches.iter_just_released() {
        println!("just released {:?}", touch);
    }

    // Just cancelled this frame
    for touch in touches.iter_just_cancelled() {
        println!("just cancelled {:?}", touch);
    }
}
```

### Touch Properties

Each touch provides:
- **Position**: Touch location on screen
- **Force**: Touch pressure (if supported by device)
- **ID**: Unique identifier for tracking multi-touch

### Raw Touch Events

You can also consume raw touch events:

```rust
fn touch_events(mut touch_events: EventReader<TouchInput>) {
    for event in touch_events.iter() {
        // Handle raw touch events
        println!("Touch event: {:?}", event);
    }
}
```

Touch input enables:
- Mobile game controls
- Tablet interfaces
- Touch-screen displays
- Multi-touch gestures

