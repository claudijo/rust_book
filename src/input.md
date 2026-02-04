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

