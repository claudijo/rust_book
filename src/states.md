# States

States organize your game into logical modes and control which systems run at any given time.

## Understanding Game States

Games naturally divide into distinct modes: loading screens, menus, gameplay, pause screens. Without states, you'd need complex boolean flags to track mode and manually enable/disable systems. States provide a cleaner abstraction that automatically manages system execution based on the current game mode.

States offer several advantages: systems run only when relevant, reducing unnecessary computation; each mode's logic is clearly separated; cleanup happens automatically when leaving a state; and the stack-based design enables natural patterns like pausing and sub-menus.

## Defining States

States are Rust enums:

```rust
#[derive(Clone, Eq, PartialEq, Debug, Hash)]
enum GameState {
    Loading,
    Menu,
    Playing,
    Paused,
}
```

## Setting Up States

Add a state to your app and configure systems for each state:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_state(GameState::Loading)
        .add_system_set(
            SystemSet::on_enter(GameState::Menu)
                .with_system(setup_menu)
        )
        .add_system_set(
            SystemSet::on_update(GameState::Menu)
                .with_system(handle_menu_input)
        )
        .add_system_set(
            SystemSet::on_exit(GameState::Menu)
                .with_system(cleanup_menu)
        )
        .run();
}
```

## State Lifecycle

Each state has three lifecycle hooks:

**on_enter** - Runs once when entering the state. Use for setup:
```rust
fn setup_game(mut commands: Commands, assets: Res<AssetServer>) {
    commands.spawn(PlayerBundle::default());
    commands.spawn(LevelBundle::from_file("level1.ron"));
}
```

**on_update** - Runs every frame while in the state. Your main game logic:
```rust
fn player_movement(
    mut query: Query<&mut Transform, With<Player>>,
    input: Res<Input<KeyCode>>
) {
    for mut transform in query.iter_mut() {
        if input.pressed(KeyCode::W) {
            transform.translation.y += 5.0;
        }
    }
}
```

**on_exit** - Runs once when leaving the state. Use for cleanup:
```rust
fn cleanup_game(mut commands: Commands, query: Query<Entity, With<GameEntity>>) {
    for entity in query.iter() {
        commands.entity(entity).despawn_recursive();
    }
}
```

## Changing States

Change state from within systems:

```rust
fn handle_menu_input(
    mut state: ResMut<State<GameState>>,
    input: Res<Input<KeyCode>>
) {
    if input.just_pressed(KeyCode::Return) {
        state.set(GameState::Playing).unwrap();
    }
}
```

## Stack-Based States

States operate as a stack, enabling powerful patterns:

**Push** a state onto the stack (keeps previous state):
```rust
state.push(GameState::Paused).unwrap();
```

**Pop** the current state (returns to previous):
```rust
state.pop().unwrap();
```

**Set** replaces the entire stack with a new state:
```rust
state.set(GameState::Menu).unwrap();
```

### Why a Stack?

The stack model enables natural patterns:

**Pause menu:**
```rust
// In game
state.push(GameState::Paused); // Game state preserved underneath

// Unpause
state.pop(); // Returns to exactly where you were
```

**Sub-menus:**
```rust
// Main menu
state.push(GameState::OptionsMenu);

// Back button
state.pop(); // Returns to main menu
```

**Modal dialogs:**
```rust
// During gameplay
state.push(GameState::ShopDialog);

// Close dialog
state.pop(); // Back to gameplay
```

## Complete Example

```rust
use bevy::prelude::*;

#[derive(Clone, Eq, PartialEq, Debug, Hash)]
enum GameState {
    MainMenu,
    Playing,
    Paused,
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_state(GameState::MainMenu)
        // Main menu
        .add_system_set(
            SystemSet::on_enter(GameState::MainMenu)
                .with_system(setup_menu)
        )
        .add_system_set(
            SystemSet::on_update(GameState::MainMenu)
                .with_system(menu_input)
        )
        // Playing
        .add_system_set(
            SystemSet::on_enter(GameState::Playing)
                .with_system(spawn_player)
                .with_system(spawn_enemies)
        )
        .add_system_set(
            SystemSet::on_update(GameState::Playing)
                .with_system(player_movement)
                .with_system(enemy_ai)
                .with_system(check_pause)
        )
        .add_system_set(
            SystemSet::on_exit(GameState::Playing)
                .with_system(save_progress)
        )
        // Paused
        .add_system_set(
            SystemSet::on_enter(GameState::Paused)
                .with_system(show_pause_menu)
        )
        .add_system_set(
            SystemSet::on_update(GameState::Paused)
                .with_system(pause_menu_input)
        )
        .run();
}

fn menu_input(mut state: ResMut<State<GameState>>, input: Res<Input<KeyCode>>) {
    if input.just_pressed(KeyCode::Space) {
        state.set(GameState::Playing).unwrap();
    }
}

fn check_pause(mut state: ResMut<State<GameState>>, input: Res<Input<KeyCode>>) {
    if input.just_pressed(KeyCode::Escape) {
        state.push(GameState::Paused).unwrap();
    }
}

fn pause_menu_input(mut state: ResMut<State<GameState>>, input: Res<Input<KeyCode>>) {
    if input.just_pressed(KeyCode::Escape) {
        state.pop().unwrap(); // Return to game
    }
    if input.just_pressed(KeyCode::Q) {
        state.set(GameState::MainMenu).unwrap(); // Exit to menu
    }
}

// Other systems omitted for brevity
```

## Common Patterns

### Loading Screen

```rust
fn check_assets_loaded(
    mut state: ResMut<State<GameState>>,
    server: Res<AssetServer>,
    handles: Res<LoadingAssets>,
) {
    if server.get_group_load_state(handles.0.iter()) == LoadState::Loaded {
        state.set(GameState::Ready).unwrap();
    }
}
```

### Level Progression

```rust
fn check_level_complete(
    mut state: ResMut<State<GameState>>,
    query: Query<&Transform, With<Player>>,
) {
    if let Ok(transform) = query.get_single() {
        if transform.translation.x > 1000.0 {
            state.set(GameState::NextLevel).unwrap();
        }
    }
}
```

### Game Over Conditions

```rust
fn check_game_over(
    mut state: ResMut<State<GameState>>,
    player: Query<&Health, With<Player>>,
    enemies: Query<Entity, With<Enemy>>,
) {
    // Player died
    if let Ok(health) = player.get_single() {
        if health.current <= 0.0 {
            state.set(GameState::GameOver).unwrap();
            return;
        }
    }
    
    // All enemies defeated
    if enemies.iter().count() == 0 {
        state.set(GameState::Victory).unwrap();
    }
}
```

## State Change Timing

State changes are queued and applied at stage boundaries. Multiple state changes in a single frame are handled sequentially, with all lifecycle systems executing before the next change.

This ensures:
- Predictable execution order
- No missed lifecycle events
- Clean state transitions

## Best Practices

**Use states for major modes** - Menu, gameplay, game over are good candidates. Don't overuse states for minor variations.

**Keep state enums simple** - Avoid deeply nested state logic. Consider separate state types for independent concerns (GameState vs MenuState).

**Clean up on exit** - Always use on_exit systems to despawn entities and free resources.

**Test state transitions** - State changes are common sources of bugs. Test all transitions.

States are fundamental to organizing game logic and are used in virtually every Bevy game.

