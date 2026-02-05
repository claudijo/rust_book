# Events

Events enable communication between systems without tight coupling. One system sends an event, and other systems can listen for and react to it.

Direct system-to-system communication creates dependencies and makes code brittle. Events solve this by providing loose coupling - the sender doesn't need to know about receivers, and receivers don't need to know about senders. This makes systems more reusable and maintainable.

Use events when multiple systems need to react to the same occurrence, when you want to decouple game logic, when actions should trigger responses in different subsystems, or when you need a broadcast mechanism.

## Basic Usage

Define an event type, register it, and use it in systems:

```rust
use bevy::prelude::*;

#[derive(Event)]
struct LevelUpEvent {
    player: Entity,
    new_level: u32,
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_event::<LevelUpEvent>()
        .add_system(gain_experience)
        .add_system(handle_level_up)
        .run();
}

fn gain_experience(
    mut commands: Commands,
    mut events: EventWriter<LevelUpEvent>,
    mut query: Query<(Entity, &mut Experience)>
) {
    for (entity, mut exp) in query.iter_mut() {
        exp.points += 10;
        
        if exp.points >= exp.next_level {
            events.send(LevelUpEvent {
                player: entity,
                new_level: exp.level + 1,
            });
        }
    }
}

fn handle_level_up(mut events: EventReader<LevelUpEvent>) {
    for event in events.iter() {
        println!("Player {:?} reached level {}!", event.player, event.new_level);
    }
}
```

## Sending Events

Use `EventWriter` to send events:

```rust
fn collision_system(
    mut collision_events: EventWriter<CollisionEvent>,
    query: Query<(Entity, &Transform, &Collider)>
) {
    for [(e1, t1, c1), (e2, t2, c2)] in query.iter_combinations() {
        if check_collision(t1, c1, t2, c2) {
            collision_events.send(CollisionEvent {
                entity_a: e1,
                entity_b: e2,
            });
        }
    }
}
```

## Reading Events

Use `EventReader` to consume events:

```rust
fn damage_system(
    mut events: EventReader<CollisionEvent>,
    mut health_query: Query<&mut Health>
) {
    for event in events.iter() {
        if let Ok(mut health) = health_query.get_mut(event.entity_a) {
            health.current -= 10.0;
        }
    }
}
```

Each `EventReader` tracks which events it has already processed. Multiple systems can read the same events independently.

## Multiple Readers

Different systems can react to the same event:

```rust
fn sound_system(mut events: EventReader<CollisionEvent>, audio: Res<Audio>) {
    for event in events.iter() {
        audio.play("collision.ogg");
    }
}

fn particles_system(
    mut events: EventReader<CollisionEvent>,
    mut commands: Commands,
    query: Query<&Transform>
) {
    for event in events.iter() {
        if let Ok(transform) = query.get(event.entity_a) {
            commands.spawn(ParticleBundle {
                transform: *transform,
                // ... particle settings
            });
        }
    }
}
```

## Event Lifecycle

Events have a two-frame lifecycle:

1. **Frame N**: Event is sent
2. **Frame N**: All EventReaders can read it
3. **Frame N+1**: Event is still available
4. **Frame N+2**: Event is cleared

This gives all systems two chances to process an event. After that, it's gone.

The two-frame buffer ensures all systems get a chance to process events regardless of system ordering. Consider two systems that might run in any order:

```rust
fn system_a(mut events: EventWriter<MyEvent>) {
    events.send(MyEvent);
}

fn system_b(mut events: EventReader<MyEvent>) {
    // Will see the event even if it runs before system_a
}
```

If `system_b` runs before `system_a` in frame N, it will catch the event in frame N+1. This design eliminates complex ordering dependencies while keeping memory usage minimal.

## Event Timing Considerations

Events don't persist across frames, so they're not suitable for:
- State that needs to persist
- Data that accumulates over time
- Information that systems might need later

Use Resources or Components for persistent data.

## Common Patterns

### Game Events

```rust
#[derive(Event)]
enum GameEvent {
    PlayerDied,
    EnemySpawned(Entity),
    LevelCompleted,
    ScoreChanged(i32),
}

fn game_over_system(mut events: EventReader<GameEvent>, mut state: ResMut<State<GameState>>) {
    for event in events.iter() {
        if let GameEvent::PlayerDied = event {
            state.set(GameState::GameOver).unwrap();
        }
    }
}
```

### Input Events

```rust
#[derive(Event)]
struct JumpEvent {
    player: Entity,
}

fn input_system(
    keyboard: Res<Input<KeyCode>>,
    mut events: EventWriter<JumpEvent>,
    query: Query<Entity, With<Player>>
) {
    if keyboard.just_pressed(KeyCode::Space) {
        for player in query.iter() {
            events.send(JumpEvent { player });
        }
    }
}

fn jump_system(
    mut events: EventReader<JumpEvent>,
    mut query: Query<&mut Velocity>
) {
    for event in events.iter() {
        if let Ok(mut velocity) = query.get_mut(event.player) {
            velocity.y = 10.0;
        }
    }
}
```

### UI Events

```rust
#[derive(Event)]
struct ButtonClickedEvent {
    button: Entity,
    button_type: ButtonType,
}

fn button_system(
    query: Query<(Entity, &Interaction, &ButtonType), Changed<Interaction>>,
    mut events: EventWriter<ButtonClickedEvent>
) {
    for (entity, interaction, button_type) in query.iter() {
        if *interaction == Interaction::Clicked {
            events.send(ButtonClickedEvent {
                button: entity,
                button_type: *button_type,
            });
        }
    }
}
```

## Built-in Events

Bevy includes several built-in event types:

- **Window events** - Resize, close, focus
- **Keyboard events** - Key presses
- **Mouse events** - Clicks, movement
- **Touch events** - Touch input
- **Asset events** - Loading, unloading

Access these with `EventReader` like custom events:

```rust
fn window_resize_system(mut events: EventReader<WindowResized>) {
    for event in events.iter() {
        println!("Window resized to {}x{}", event.width, event.height);
    }
}
```

## Performance

Events are efficient:
- `EventWriter` is essentially free (adds to a Vec)
- `EventReader` tracks an index, no allocation
- Event storage is reused across frames
- Clearing old events is O(1) buffer swap

The system automatically manages event buffers, swapping them each frame.

## Best Practices

**Keep events small** - Events are copied, not referenced. Large data should use entities and components.

**Use enums for related events** - Group similar events together for easier handling.

**Don't overuse events** - For direct data flow, consider system chaining. For persistent state, use resources.

**Name events clearly** - Event names should describe what happened, not what should happen next.

Events provide a clean, efficient way to build loosely coupled systems that react to game events without creating tangled dependencies.

