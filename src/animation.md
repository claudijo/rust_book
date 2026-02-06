# Animation

Bevy brings 3D models to life through skeletal animation. Characters can walk, run, jump, and perform complex movements by animating their skeleton over time.

## Understanding Animation

Animation in 3D games works by transforming bones in a skeleton over time. A character model has an internal skeleton - a hierarchy of bones. Each bone influences nearby vertices in the mesh. By moving bones, the mesh deforms naturally, creating realistic motion.

An animation clip stores these bone transformations as keyframes over time. The animation system interpolates between keyframes to create smooth motion. Multiple animations (walk, run, idle) can be swapped or blended to create responsive character behavior.

## AnimationPlayer Component

The `AnimationPlayer` component controls animation playback. Add it to an entity with an animated model:

```rust
fn spawn_character(
    mut commands: Commands,
    asset_server: Res<AssetServer>
) {
    commands.spawn((
        SceneBundle {
            scene: asset_server.load("character.gltf#Scene0"),
            ..Default::default()
        },
        AnimationPlayer::default(),
    ));
}
```

AnimationPlayer can play, pause, seek, and control animation clips.

## Animation Clips

Animation clips are assets containing keyframe data. Load them like any other asset:

```rust
#[derive(Resource)]
struct CharacterAnimations {
    idle: Handle<AnimationClip>,
    walk: Handle<AnimationClip>,
    run: Handle<AnimationClip>,
}

fn load_animations(
    mut commands: Commands,
    asset_server: Res<AssetServer>
) {
    commands.insert_resource(CharacterAnimations {
        idle: asset_server.load("character.gltf#Animation0"),
        walk: asset_server.load("character.gltf#Animation1"),
        run: asset_server.load("character.gltf#Animation2"),
    });
}
```

GLTF files can contain multiple animation clips. Use the `#AnimationN` label to load specific animations.

## Playing Animations

Start playing an animation:

```rust
fn start_walking(
    animations: Res<CharacterAnimations>,
    mut players: Query<&mut AnimationPlayer>
) {
    for mut player in players.iter_mut() {
        player.start(animations.walk.clone());
    }
}
```

The animation begins immediately and plays once by default. Use `start()` to begin or restart an animation.

## Looping

Make animations repeat continuously:

```rust
fn start_idle_animation(
    animations: Res<CharacterAnimations>,
    mut players: Query<&mut AnimationPlayer>
) {
    for mut player in players.iter_mut() {
        player.start(animations.idle.clone()).repeat();
    }
}
```

Looping animations work perfectly for idle states, walking, running, or any cyclic motion.

## Animation Control

The release notes mention that animations can be "paused, scrubbed, looped, reversed, and speed controlled" using AnimationPlayer. The specific control API includes:

```rust
// Start/restart an animation
player.start(animation_handle.clone());

// Loop an animation
player.start(animation_handle.clone()).repeat();

// Play without overwriting (0.9 behavior)
player.play(animation_handle.clone()).repeat();
```

Use `start()` to begin or restart an animation. Use `play()` if you don't want to interrupt an already-playing animation. Additional control methods (pause, resume, seek, speed) are available - consult the Bevy API documentation for complete AnimationPlayer methods.

## Loading Animations from GLTF

GLTF files often contain both models and animations. Load the complete scene and animations separately:

```rust
fn setup(
    mut commands: Commands,
    asset_server: Res<AssetServer>
) {
    // Spawn the model
    commands.spawn(SceneBundle {
        scene: asset_server.load("fox.gltf#Scene0"),
        ..Default::default()
    });
    
    // Load animation clips
    commands.insert_resource(FoxAnimations {
        walk: asset_server.load("fox.gltf#Animation0"),
        run: asset_server.load("fox.gltf#Animation1"),
        idle: asset_server.load("fox.gltf#Animation2"),
    });
}
```

Then play animations when the AnimationPlayer is ready:

```rust
fn play_on_spawn(
    animations: Res<FoxAnimations>,
    mut players: Query<&mut AnimationPlayer, Added<AnimationPlayer>>
) {
    for mut player in players.iter_mut() {
        player.start(animations.walk.clone()).repeat();
    }
}
```

The `Added<AnimationPlayer>` filter ensures this runs only for newly spawned players.

## Animating Transforms

AnimationPlayer isn't limited to skeletal animation. The release notes mention it can animate arbitrary Transform components, enabling moving platforms, rotating objects, or cutscene cameras. Animations are loaded from GLTF files which can contain transform animations in addition to skeletal animations.

## Character State Management

Track animation state with components:

```rust
#[derive(Component)]
enum CharacterState {
    Idle,
    Walking,
    Running,
    Jumping,
}

fn update_character_animation(
    animations: Res<CharacterAnimations>,
    mut query: Query<(&CharacterState, &mut AnimationPlayer), Changed<CharacterState>>
) {
    for (state, mut player) in query.iter_mut() {
        let animation = match state {
            CharacterState::Idle => &animations.idle,
            CharacterState::Walking => &animations.walk,
            CharacterState::Running => &animations.run,
            CharacterState::Jumping => &animations.jump,
        };
        
        player.start(animation.clone()).repeat();
    }
}
```

This pattern switches animations based on game state changes.

## Best Practices

**Preload animations** - Load animation clips during initialization or loading screens. Don't wait until you need to play them.

**Use state machines** - For complex characters, implement state machines to manage animation transitions cleanly.

**Cache animation handles** - Store animation handles in resources or components rather than loading repeatedly.

**Test animation timing** - Animation speeds affect gameplay feel. Playtest and adjust speeds to match your game's responsiveness.

**Name animations clearly** - Use descriptive names for animation clips. Future you will thank present you.

**Consult API docs** - The release notes mention features like pausing, scrubbing, reversing, and speed control. Check the Bevy API documentation for the complete AnimationPlayer interface.

Bevy's animation system provides the foundation for bringing 3D worlds to life. As the system evolves, expect more sophisticated features like blending, inverse kinematics, and animation graphs.



