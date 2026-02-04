# States

**Added in Bevy 0.4**

Bevy States allow you to organize your app into logical states and enable/disable systems according to the current state.

## Defining States

States are defined as normal Rust enums:

```rust
#[derive(Clone, Eq, PartialEq, Debug, Hash)]
enum AppState {
    Loading,
    Menu,
    InGame,
    Paused,
}
```

## Adding States to Your App

Add a State resource with a default value:

```rust
fn main() {
    App::build()
        .add_plugins(DefaultPlugins)
        .add_resource(State::new(AppState::Loading))
        .add_stage_after(stage::UPDATE, "game_state", StateStage::<AppState>::default())
        .run();
}
```

The `StateStage` is where state lifecycle systems run.

## State Lifecycle Events

Systems can be registered for different lifecycle events:

```rust
app
    .on_state_enter("game_state", AppState::Menu, setup_menu.system())
    .on_state_update("game_state", AppState::Menu, menu_system.system())
    .on_state_exit("game_state", AppState::Menu, cleanup_menu.system())
    .on_state_enter("game_state", AppState::InGame, setup_game.system())
    .on_state_update("game_state", AppState::InGame, game_play.system());
```

### Lifecycle Events

- **on_enter**: Runs once when first entering a state
- **on_update**: Runs every frame while in this state (after enter/exit)
- **on_exit**: Runs once when exiting a state

## Changing States

Queue a state change from within a system:

```rust
fn menu_system(
    mut state: ResMut<State<AppState>>,
    keyboard: Res<Input<KeyCode>>
) {
    if keyboard.just_pressed(KeyCode::Return) {
        state.set_next(AppState::InGame).unwrap();
    }
}
```

## State Change Behavior

State changes are queued and applied at the end of the StateStage:

1. **Same-frame execution**: If you change state within a StateStage, lifecycle events occur in the same update/frame
2. **Multiple changes**: You can change state multiple times, and the system will continue running lifecycle events until no more changes are queued
3. **Guaranteed execution**: Multiple state changes can be applied within a single frame

This ensures smooth state transitions and predictable behavior.

## Example: Game with Menu

```rust
#[derive(Clone, Eq, PartialEq, Debug, Hash)]
enum GameState {
    Menu,
    Playing,
    Paused,
}

fn main() {
    App::build()
        .add_plugins(DefaultPlugins)
        .add_resource(State::new(GameState::Menu))
        .add_stage_after(stage::UPDATE, "game", StateStage::<GameState>::default())
        // Menu state
        .on_state_enter("game", GameState::Menu, setup_menu.system())
        .on_state_update("game", GameState::Menu, menu_ui.system())
        .on_state_exit("game", GameState::Menu, cleanup_menu.system())
        // Playing state
        .on_state_enter("game", GameState::Playing, setup_level.system())
        .on_state_update("game", GameState::Playing, player_movement.system())
        .on_state_update("game", GameState::Playing, enemy_ai.system())
        .on_state_exit("game", GameState::Playing, save_progress.system())
        // Paused state
        .on_state_enter("game", GameState::Paused, show_pause_menu.system())
        .on_state_update("game", GameState::Paused, pause_menu_input.system())
        .on_state_exit("game", GameState::Paused, hide_pause_menu.system())
        .run();
}

fn menu_ui(mut state: ResMut<State<GameState>>, input: Res<Input<KeyCode>>) {
    if input.just_pressed(KeyCode::Space) {
        state.set_next(GameState::Playing).unwrap();
    }
}

fn player_movement(mut state: ResMut<State<GameState>>, input: Res<Input<KeyCode>>) {
    if input.just_pressed(KeyCode::Escape) {
        state.set_next(GameState::Paused).unwrap();
    }
    
    // Player movement logic...
}

fn pause_menu_input(mut state: ResMut<State<GameState>>, input: Res<Input<KeyCode>>) {
    if input.just_pressed(KeyCode::Escape) {
        state.set_next(GameState::Playing).unwrap();
    }
    if input.just_pressed(KeyCode::Q) {
        state.set_next(GameState::Menu).unwrap();
    }
}
```

## Common Use Cases

### Loading Screen

```rust
enum GameState {
    Loading,
    Ready,
}

fn check_assets_loaded(
    mut state: ResMut<State<GameState>>,
    asset_server: Res<AssetServer>,
    handles: Res<MyAssetHandles>,
) {
    if asset_server.get_load_state(handles.texture) == LoadState::Loaded {
        state.set_next(GameState::Ready).unwrap();
    }
}
```

### Level Transitions

```rust
enum GameState {
    Level1,
    Level2,
    Level3,
}

fn check_level_complete(
    mut state: ResMut<State<GameState>>,
    player: Query<&Player>,
) {
    if player.iter().next().map(|p| p.at_exit()).unwrap_or(false) {
        state.set_next(GameState::Level2).unwrap();
    }
}
```

### Game Over / Win Conditions

```rust
enum GameState {
    Playing,
    GameOver,
    Victory,
}

fn check_win_condition(
    mut state: ResMut<State<GameState>>,
    enemies: Query<&Enemy>,
    player: Query<&Player>,
) {
    if enemies.iter().count() == 0 {
        state.set_next(GameState::Victory).unwrap();
    }
    
    if player.iter().next().map(|p| p.health <= 0).unwrap_or(false) {
        state.set_next(GameState::GameOver).unwrap();
    }
}
```

## Benefits

- **Clear organization**: Game logic naturally grouped by state
- **Automatic cleanup**: on_exit systems handle cleanup automatically
- **Performance**: Only run systems relevant to current state
- **Testability**: Each state can be tested independently
- **Maintainability**: Easy to see what happens in each state

States are a powerful tool for organizing complex game logic and are commonly used in almost every Bevy game.

