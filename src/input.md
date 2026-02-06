# Input

Games respond to player actions through input. Bevy provides a unified input system that works across platforms - keyboard, mouse, gamepads, and touch all use consistent APIs.

## Keyboard Input

The `Input<KeyCode>` resource tracks which keys are pressed. Query it in any system to respond to keyboard input:

```rust
fn player_movement(
    keyboard: Res<Input<KeyCode>>,
    mut query: Query<&mut Transform, With<Player>>
) {
    for mut transform in query.iter_mut() {
        if keyboard.pressed(KeyCode::W) {
            transform.translation.y += 5.0;
        }
        if keyboard.pressed(KeyCode::S) {
            transform.translation.y -= 5.0;
        }
        if keyboard.pressed(KeyCode::A) {
            transform.translation.x -= 5.0;
        }
        if keyboard.pressed(KeyCode::D) {
            transform.translation.x += 5.0;
        }
    }
}
```

The input system tracks three states for each key:

**pressed()** - Key is currently held down. Use for continuous actions like movement.

**just_pressed()** - Key was pressed this frame. Use for discrete actions like jumping or shooting.

**just_released()** - Key was released this frame. Use for actions that occur on release.

```rust
fn handle_actions(keyboard: Res<Input<KeyCode>>, mut events: EventWriter<JumpEvent>) {
    if keyboard.just_pressed(KeyCode::Space) {
        events.send(JumpEvent);
    }
}
```

## Alternative Key Bindings

Often you want multiple keys to trigger the same action. The `any_pressed` method checks if any key in a set is pressed:

```rust
fn movement(keyboard: Res<Input<KeyCode>>, mut query: Query<&mut Transform>) {
    for mut transform in query.iter_mut() {
        // Move up with W or Up arrow
        if keyboard.any_pressed([KeyCode::W, KeyCode::Up]) {
            transform.translation.y += 5.0;
        }
        
        // Move left with A or Left arrow
        if keyboard.any_pressed([KeyCode::A, KeyCode::Left]) {
            transform.translation.x -= 5.0;
        }
    }
}
```

## Inspecting Pressed Keys

Sometimes you need to know which keys are currently pressed:

```rust
fn debug_input(keyboard: Res<Input<KeyCode>>) {
    for key in keyboard.get_pressed() {
        println!("Currently pressed: {:?}", key);
    }
    
    for key in keyboard.get_just_pressed() {
        println!("Just pressed this frame: {:?}", key);
    }
    
    for key in keyboard.get_just_released() {
        println!("Just released this frame: {:?}", key);
    }
}
```

These iterators are useful for implementing text input, debug consoles, or key rebinding systems.

## Mouse Input

Mouse buttons work identically to keyboard keys:

```rust
fn shooting(
    mouse: Res<Input<MouseButton>>,
    mut commands: Commands
) {
    if mouse.just_pressed(MouseButton::Left) {
        // Fire weapon
        commands.spawn(BulletBundle::default());
    }
    
    if mouse.pressed(MouseButton::Right) {
        // Aim down sights
    }
}
```

Common mouse buttons include `Left`, `Right`, `Middle`, and additional numbered buttons for gaming mice.

## Mouse Position

While `Input<MouseButton>` tracks button state, you need `Window` to get cursor position:

```rust
fn cursor_position(
    windows: Res<Windows>,
    camera_query: Query<(&Camera, &GlobalTransform)>
) {
    let window = windows.get_primary().unwrap();
    
    if let Some(cursor_pos) = window.cursor_position() {
        println!("Cursor at: {:?}", cursor_pos);
        
        // Convert to world coordinates if needed
        // (requires camera transform calculation)
    }
}
```

## Gamepad Input

Gamepads work like keyboards but require iteration over connected controllers. The `Gamepads` resource tracks which controllers are plugged in:

```rust
fn gamepad_input(
    gamepads: Res<Gamepads>,
    button_inputs: Res<Input<GamepadButton>>,
    mut query: Query<&mut Transform, With<Player>>
) {
    for gamepad in gamepads.iter() {
        if button_inputs.just_pressed(GamepadButton::new(gamepad, GamepadButtonType::South)) {
            println!("Player {} pressed A/Cross", gamepad.id);
        }
        
        if button_inputs.pressed(GamepadButton::new(gamepad, GamepadButtonType::West)) {
            println!("Player {} holding X/Square", gamepad.id);
        }
    }
}
```

Gamepad button types use generic names that map appropriately across controllers:
- `South` - A on Xbox, Cross on PlayStation
- `East` - B on Xbox, Circle on PlayStation  
- `West` - X on Xbox, Square on PlayStation
- `North` - Y on Xbox, Triangle on PlayStation

## Analog Sticks and Triggers

Gamepad axes provide analog input values between -1.0 and 1.0:

```rust
fn gamepad_movement(
    gamepads: Res<Gamepads>,
    axes: Res<Axis<GamepadAxis>>,
    mut query: Query<&mut Transform, With<Player>>
) {
    for gamepad in gamepads.iter() {
        let left_x = axes
            .get(GamepadAxis::new(gamepad, GamepadAxisType::LeftStickX))
            .unwrap_or(0.0);
        let left_y = axes
            .get(GamepadAxis::new(gamepad, GamepadAxisType::LeftStickY))
            .unwrap_or(0.0);
        
        for mut transform in query.iter_mut() {
            transform.translation.x += left_x * 5.0;
            transform.translation.y += left_y * 5.0;
        }
    }
}
```

Triggers return values from 0.0 (released) to 1.0 (fully pressed).

## Configuring Deadz ones

Analog sticks aren't perfectly centered when released. Deadzones filter out small values to prevent drift:

```rust
fn setup_gamepad(mut settings: ResMut<GamepadSettings>) {
    settings.default_axis_settings.set_deadzone_lowerbound(-0.1);
    settings.default_axis_settings.set_deadzone_upperbound(0.1);
    
    // Or configure per-controller, per-axis
    settings.axis_settings.insert(
        GamepadAxis::new(Gamepad::new(0), GamepadAxisType::LeftStickX),
        AxisSettings::new(-0.15, 0.15, -0.9, 0.9),
    );
}
```

Set deadzones based on your game's needs. Racing games might want smaller deadzones for precision, while platformers might use larger ones for reliable movement.

## Touch Input

Touch input works on mobile devices and touch screens. The `Touches` resource tracks all active touches:

```rust
fn handle_touches(touches: Res<Touches>) {
    for touch in touches.iter() {
        let position = touch.position();
        println!("Touch at: {:?}", position);
        
        if let Some(force) = touch.force() {
            println!("Pressure: {}", force);
        }
    }
}
```

Touch input distinguishes between touch lifecycle events:

```rust
fn touch_events(touches: Res<Touches>) {
    for touch in touches.iter_just_pressed() {
        println!("New touch started at {:?}", touch.position());
    }
    
    for touch in touches.iter_just_released() {
        println!("Touch ended at {:?}", touch.position());
    }
    
    for touch in touches.iter_just_cancelled() {
        println!("Touch cancelled");
    }
}
```

Each touch has a unique ID, enabling multi-touch tracking:

```rust
fn track_gestures(touches: Res<Touches>, mut state: ResMut<GestureState>) {
    if touches.iter().count() == 2 {
        let mut iter = touches.iter();
        let touch1 = iter.next().unwrap();
        let touch2 = iter.next().unwrap();
        
        let distance = touch1.position().distance(touch2.position());
        state.pinch_distance = Some(distance);
    }
}
```

## Input Events

For more control, consume raw input events:

```rust
fn keyboard_events(mut events: EventReader<KeyboardInput>) {
    for event in events.iter() {
        println!("Key event: {:?}", event);
    }
}

fn mouse_events(mut events: EventReader<MouseButtonInput>) {
    for event in events.iter() {
        println!("Mouse event: {:?}", event);
    }
}
```

Events provide lower-level access, useful for implementing text input or custom input systems.

## Cross-Platform Input

The input system abstracts platform differences. The same code handles keyboard on desktop, gamepad on console, and touch on mobile. When porting to new platforms, input code often requires minimal changes.

For maximum compatibility, provide multiple input methods. Support both keyboard and gamepad for desktop games. Include touch controls as an option even on platforms with physical inputs.

## Best Practices

**Use just_pressed for actions** - Button presses, menu selections, and one-time triggers should use `just_pressed` to avoid repeated activation.

**Use pressed for continuous actions** - Movement, aiming, and held actions should use `pressed` to work smoothly while the input is held.

**Support multiple input methods** - Let players choose between keyboard, gamepad, or touch based on preference.

**Configure deadzones appropriately** - Test analog inputs with different controllers to find good deadzone values.

**Provide key rebinding** - Let players customize their controls for accessibility and preference.

Bevy's input system provides everything you need for responsive player controls across all platforms.

