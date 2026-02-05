# Bevy ECS

All Bevy engine and game logic is built on a custom Entity Component System (ECS).

## Understanding ECS

Entity Component Systems separate data from behavior in a way that maximizes performance and flexibility.

**Components** are pure data structures. A Position component holds coordinates. A Velocity component holds speed. A Health component holds hit points.

**Entities** are unique identifiers that group components together. Entity #42 might have Position, Velocity, and Health components, making it a moving, damageable object.

**Systems** are functions that operate on entities with specific components. A movement system processes all entities that have both Position and Velocity, updating positions based on velocities.

This separation encourages clean designs by forcing you to think in terms of data and transformations, not object hierarchies.

## Why ECS?

Traditional object-oriented game code often becomes a tangled mess of inheritance hierarchies and cross-dependencies. ECS solves this by:

**Composition over inheritance** - Build complex entities by combining simple components rather than fighting with inheritance trees.

**Cache-friendly iteration** - Components of the same type are stored contiguously in memory. When iterating over thousands of entities, this locality means your CPU can prefetch data efficiently instead of jumping around memory randomly.

**Automatic parallelization** - Since systems declare their data dependencies explicitly, Bevy can safely run systems in parallel without locks or synchronization overhead.

**Clean separation** - Data (components) is completely separate from logic (systems), making both easier to understand, test, and modify.

## Components

Components are ordinary Rust structs that derive the `Component` trait:

```rust
#[derive(Component)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(Component)]
struct Velocity {
    x: f32,
    y: f32,
}

#[derive(Component)]
struct Health {
    current: f32,
    max: f32,
}
```

### Component Storage

Bevy offers two storage strategies optimized for different access patterns:

**Table storage** (default) - Components are stored in contiguous arrays. Fast to iterate, slower to add/remove.

```rust
#[derive(Component)]
struct Position { x: f32, y: f32 }
```

**SparseSet storage** - Components are stored in a sparse set structure. Fast to add/remove, slightly slower to iterate.

```rust
#[derive(Component)]
#[component(storage = "SparseSet")]
struct Marker;
```

Use SparseSet for components that are frequently added and removed (like temporary status effects) or accessed rarely. The default Table storage is best for components that exist throughout an entity's lifetime and are accessed frequently.

## Systems

Systems are normal Rust functions that operate on the game world:

```rust
fn movement(mut query: Query<(&mut Position, &Velocity)>) {
    for (mut position, velocity) in query.iter_mut() {
        position.x += velocity.x;
        position.y += velocity.y;
    }
}
```

The function signature tells Bevy everything it needs to know. From the parameters, Bevy:
- Determines what data this system needs
- Schedules it to run in parallel with other systems when safe
- Provides access only to the requested data
- Tracks changes automatically for optimization

## Complete Example

```rust
use bevy::prelude::*;

#[derive(Component)]
struct Velocity { x: f32, y: f32 }

#[derive(Component)]
struct Position { x: f32, y: f32 }

fn spawn_entities(mut commands: Commands) {
    commands.spawn((
        Position { x: 0.0, y: 0.0 },
        Velocity { x: 1.0, y: 0.5 },
    ));
}

fn movement(mut query: Query<(&mut Position, &Velocity)>) {
    for (mut position, velocity) in query.iter_mut() {
        position.x += velocity.x;
        position.y += velocity.y;
    }
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_startup_system(spawn_entities)
        .add_system(movement)
        .run();
}
```

This complete application spawns moving entities with automatic parallel scheduling and change detection.

## Queries

Queries are how systems access entity data. A query specifies what components you need, and Bevy efficiently provides all matching entities.

### Basic Queries

```rust
fn movement(mut query: Query<(&mut Position, &Velocity)>) {
    for (mut position, velocity) in query.iter_mut() {
        position.x += velocity.x;
        position.y += velocity.y;
    }
}
```

The query `Query<(&mut Position, &Velocity)>` means: "Give me mutable access to Position and immutable access to Velocity for all entities that have both components."

### Iteration

Mutable iteration when you need to modify components:

```rust
fn system(mut query: Query<&mut Position>) {
    for mut position in query.iter_mut() {
        position.x += 1.0;
    }
}
```

Immutable iteration when you only need to read:

```rust
fn system(query: Query<(&Position, &Velocity)>) {
    for (position, velocity) in query.iter() {
        println!("Entity at {:?} moving {:?}", position, velocity);
    }
}
```

You can iterate immutably even with mutable access:

```rust
fn system(mut query: Query<&mut Health>) {
    // Check first without modifying
    for health in query.iter() {
        if health.current <= 0.0 {
            println!("Found dead entity");
        }
    }
    
    // Then modify
    for mut health in query.iter_mut() {
        health.current = health.current.min(health.max);
    }
}
```

### Accessing Specific Entities

```rust
fn system(query: Query<(&Position, &Velocity)>, target: Res<TargetEntity>) {
    if let Ok((position, velocity)) = query.get(target.0) {
        println!("Target at {:?} moving {:?}", position, velocity);
    }
}
```

### Single Entity Queries

When you expect exactly one matching entity:

```rust
fn system(query: Query<&Transform, With<Player>>) {
    let player_transform = query.single();
    // Use the transform
}
```

This panics if there isn't exactly one match. For fallible access:

```rust
fn system(query: Query<&Transform, With<Player>>) {
    if let Ok(player_transform) = query.get_single() {
        // Use the transform
    }
}
```

### Query Filters

Filters let you specify additional constraints without retrieving components:

```rust
fn system(query: Query<(&Transform, &Velocity), (With<Enemy>, Without<Dead>)>) {
    for (transform, velocity) in query.iter() {
        // Only processes living enemies
    }
}
```

Common filters:
- `With<T>` - Entity must have component T
- `Without<T>` - Entity must not have component T
- `Changed<T>` - Component T was added or modified this frame
- `Added<T>` - Component T was just added

You can create type aliases for reusable filter combinations:

```rust
type Damageable = (With<Health>, Without<Invulnerable>);

fn damage_system(query: Query<&mut Health, Damageable>) {
    // Only entities that can be damaged
}
```

### Change Detection

Change detection lets you react only when components are modified:

```rust
fn react_to_movement(query: Query<&Transform, Changed<Transform>>) {
    for transform in query.iter() {
        println!("Entity moved to {:?}", transform.translation);
    }
}
```

Track when components are removed:

```rust
fn cleanup_removed(mut query: Query<&Position>) {
    for entity in query.removed::<Weapon>() {
        println!("Entity {:?} lost its weapon", entity);
    }
}
```

### Iterating Combinations

Check all pairs of entities (useful for collision detection):

```rust
fn collision_system(query: Query<(Entity, &Transform, &Collider)>) {
    for [(e1, t1, c1), (e2, t2, c2)] in query.iter_combinations() {
        if check_collision(t1, c1, t2, c2) {
            println!("Collision between {:?} and {:?}", e1, e2);
        }
    }
}
```

Each pair is checked exactly once. Note that this is O(n²) - use spatial partitioning for large numbers of entities.

### Parallel Queries

Distribute query work across multiple threads:

```rust
fn system(pool: Res<ComputeTaskPool>, mut query: Query<&mut Transform>) {
    query.par_iter_mut().for_each(&pool, |mut transform| {
        // This runs in parallel across thread pool
        transform.translation.y -= 9.8 * delta;
    });
}
```

Bevy automatically breaks the query into batches and distributes them across available threads.

### Multiple Queries

Systems can have multiple independent queries:

```rust
fn system(
    players: Query<&Transform, With<Player>>,
    enemies: Query<&Transform, With<Enemy>>
) {
    for player_pos in players.iter() {
        for enemy_pos in enemies.iter() {
            let distance = player_pos.translation.distance(enemy_pos.translation);
            if distance < 5.0 {
                println!("Enemy nearby!");
            }
        }
    }
}
```

### Conflicting Queries

If you need overlapping mutable access, use QuerySet:

```rust
fn system(mut queries: QuerySet<(
    QueryState<&mut Health>,
    QueryState<(&mut Health, &Armor)>
)>) {
    // First query
    for health in queries.q0().iter() {
        println!("Health: {}", health.current);
    }
    
    // Second query (overlaps with first)
    for (mut health, armor) in queries.q1().iter_mut() {
        health.current += armor.regeneration;
    }
}
```

QuerySet ensures only one query is active at a time, preventing unsafe mutable aliasing while still allowing both queries in the same system.

## Resources

Resources are global data accessible to all systems:

```rust
fn system(time: Res<Time>, mut score: ResMut<Score>) {
    score.points += time.delta_seconds() as i32;
}
```

Use `Res<T>` for immutable access and `ResMut<T>` for mutable access.

### Local Resources

Local resources are unique per-system instance. They're initialized to their default value:

```rust
#[derive(Default)]
struct FrameCount(u32);

fn system(mut count: Local<FrameCount>, query: Query<&Position>) {
    count.0 += 1;
    println!("System has run {} times", count.0);
}
```

Different instances of the same system each have their own local state.

### SystemState

Access system parameters directly without creating a full system:

```rust
let mut system_state: SystemState<(Res<AssetServer>, Query<&Transform>)> = 
    SystemState::new(&mut world);

let (asset_server, query) = system_state.get(&world);

for transform in query.iter() {
    // Use transform
}
```

This is useful for:
- Direct world manipulation outside systems
- Tools and editor code
- Custom execution patterns

SystemState caches internal data like regular systems, making repeated access fast.

## Commands

Commands queue world changes that are applied at the end of each stage:

```rust
fn spawn_enemies(mut commands: Commands) {
    commands.spawn((
        Enemy,
        Health { current: 100.0, max: 100.0 },
        Transform::from_xyz(10.0, 0.0, 0.0),
    ));
}
```

### Spawning Entities

```rust
fn system(mut commands: Commands) {
    // Spawn with a bundle
    let entity = commands.spawn_bundle(EnemyBundle::default()).id();
    
    // Spawn and add components incrementally
    commands.spawn()
        .insert_bundle((Position::default(), Velocity::default()))
        .insert(Enemy);
}
```

### Modifying Entities

```rust
fn system(mut commands: Commands, query: Query<Entity, With<Defeated>>) {
    for entity in query.iter() {
        commands.entity(entity)
            .insert(Despawning)
            .remove::<Enemy>();
    }
}
```

### Despawning Entities

```rust
fn cleanup(mut commands: Commands, query: Query<Entity, With<Despawning>>) {
    for entity in query.iter() {
        commands.entity(entity).despawn();
    }
}
```

### Hierarchy Commands

```rust
// Despawn an entity and all its descendants
commands.entity(parent).despawn_descendants();

// Remove specific children
commands.entity(parent).remove_children(&[child1, child2]);
```

### Why Commands?

Commands are deferred because immediate changes would invalidate iterators and cause race conditions in parallel systems. By queuing changes and applying them at stage boundaries, Bevy maintains safety and performance.

## System Parameters

Systems can accept any type implementing `SystemParam`:

- `Query<T>` - Access entities with specific components
- `Res<T>`, `ResMut<T>` - Access global resources
- `Commands` - Queue world changes
- `Local<T>` - Per-system state
- `EventReader<T>`, `EventWriter<T>` - Event handling
- `Option<Res<T>>` - Optional resources
- `Res<Assets<T>>` - Access asset collections

Parameters can be in any order:

```rust
fn system(
    query: Query<&Transform>,
    mut commands: Commands,
    time: Res<Time>,
    assets: Res<Assets<Mesh>>
) {
    // All valid regardless of order
}
```

## System Chaining

Systems can have inputs and outputs, allowing data flow between systems:

```rust
fn producer(query: Query<&Transform>) -> Vec<Vec3> {
    query.iter().map(|t| t.translation).collect()
}

fn consumer(In(positions): In<Vec<Vec3>>) {
    for pos in positions {
        println!("Position: {:?}", pos);
    }
}

fn main() {
    App::new()
        .add_system(producer.chain(consumer))
        .run();
}
```

This is useful for:
- Error handling pipelines
- Data transformation chains
- Composing reusable system components

## Sub Apps

Large applications can be split into multiple sub-apps, each with their own schedule:

```rust
#[derive(AppLabel)]
struct RenderApp;

let mut render_app = App::empty();

app.add_sub_app(RenderApp, render_app, |app_world, render_app| {
    // Execute render logic
});
```

Bevy's renderer runs as a sub-app, maintaining strict separation between simulation and rendering. This enables features like pipelined rendering where the next frame's simulation runs while the current frame renders.

## Performance

Bevy ECS is designed for high performance:

**Cache-friendly iteration** - Components are packed tightly in arrays. Iterating thousands of entities benefits from CPU cache prefetching.

**Lock-free parallelism** - Systems run concurrently without atomic operations or locks. Bevy's scheduler uses compile-time analysis of system parameters to ensure safety.

**Minimal overhead** - Queries compile to direct memory access. No virtual dispatch, no dynamic allocations in hot paths.

**Generational entity IDs** - Entities use array indices for O(1) lookups instead of hash maps.

According to ecs_bench benchmarks, Bevy ECS is among the fastest Rust ECS implementations for iteration, spawning, and component access.

