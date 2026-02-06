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

### Newtype Components

Wrap types in components using the newtype pattern with Deref/DerefMut for ergonomic access:

```rust
#[derive(Component, Deref, DerefMut)]
struct Score(i32);

#[derive(Component, Deref, DerefMut)]
struct Items(Vec<Item>);

fn update_score(mut query: Query<&mut Score>) {
    for mut score in query.iter_mut() {
        // Access the inner i32 directly through deref
        **score += 10;
        
        // Or use deref coercion
        if *score > 100 {
            println!("High score!");
        }
    }
}

fn manage_inventory(mut query: Query<&mut Items>) {
    for mut items in query.iter_mut() {
        // Vec methods available directly
        items.push(Item::new("Sword"));
        items.retain(|item| !item.is_broken());
    }
}
```

The `Deref` and `DerefMut` derives provide transparent access to the wrapped type, eliminating the need for `.0` field access. This keeps public APIs clean while maintaining type safety.

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

### Ergonomic Iteration Syntax

Queries implement `IntoIterator`, enabling cleaner iteration syntax:

```rust
fn system(mut players: Query<&mut Player>) {
    // Instead of players.iter()
    for player in &players {
        println!("Player: {}", player.name);
    }
    
    // Instead of players.iter_mut()
    for mut player in &mut players {
        player.score += 1;
    }
}
```

Use `&query` for read-only iteration and `&mut query` for mutable iteration. Both approaches work - choose whichever feels more natural for your code.

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

### AnyOf Queries

Query entities that have any of several components:

```rust
fn system(query: Query<AnyOf<(&Player, &Enemy)>>) {
    for (player, enemy) in query.iter() {
        // At least one is guaranteed to be Some
        if let Some(player) = player {
            println!("Found player");
        }
        if let Some(enemy) = enemy {
            println!("Found enemy");
        }
    }
}
```

`AnyOf` returns entities with Player but not Enemy, Enemy but not Player, or both. Each component in the tuple is wrapped in `Option`. This is useful for systems that handle multiple related types:

```rust
fn render_sprites(
    query: Query<(
        &Transform,
        AnyOf<(&Sprite, &AnimatedSprite, &TiledSprite)>
    )>
) {
    for (transform, (sprite, animated, tiled)) in query.iter() {
        // Render whichever sprite type exists
        if let Some(sprite) = sprite {
            render_static_sprite(sprite, transform);
        } else if let Some(animated) = animated {
            render_animated_sprite(animated, transform);
        } else if let Some(tiled) = tiled {
            render_tiled_sprite(tiled, transform);
        }
    }
}
```

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

### Custom Query Types

Create reusable query bundles with WorldQuery derives:

```rust
#[derive(WorldQuery)]
#[world_query(mutable)]
struct PlayerQuery<'w> {
    transform: &'w mut Transform,
    velocity: &'w mut Velocity,
    health: &'w mut Health,
    stats: &'w PlayerStats,
}

fn move_players(mut players: Query<PlayerQuery>) {
    for mut player in players.iter_mut() {
        // Access all components through the player bundle
        player.velocity.x += player.stats.speed;
        player.transform.translation.x += player.velocity.x;
        
        if player.health.current <= 0.0 {
            // Handle death
        }
    }
}

fn player_collision(mut players: Query<(Entity, PlayerQuery)>) {
    for (entity, mut player) in players.iter_mut() {
        // Entity ID available alongside components
    }
}
```

The `#[world_query(mutable)]` attribute enables mutable access. Custom queries reduce repetition and make code more maintainable when the same component sets appear across multiple systems.

Read-only queries don't need the mutable attribute:

```rust
#[derive(WorldQuery)]
struct EnemyQuery<'w> {
    transform: &'w Transform,
    ai: &'w AI,
    health: &'w Health,
}
```

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

### Querying Specific Entities

Access multiple specific entities from a query:

```rust
#[derive(Resource)]
struct ImportantEntities {
    player: Entity,
    target: Entity,
    camera: Entity,
}

fn system(
    entities: Res<ImportantEntities>,
    mut query: Query<&mut Transform>
) {
    // Query multiple entities at once
    let [player_transform, target_transform, camera_transform] = 
        query.many_mut([entities.player, entities.target, entities.camera]);
    
    // Now we have mutable access to all three simultaneously
    player_transform.translation.x += 1.0;
    camera_transform.look_at(target_transform.translation, Vec3::Y);
}
```

`many_mut()` returns an array of mutable references, avoiding borrow checker conflicts. It panics if entities don't exist or overlap. Use `get_many_mut()` for a Result-based variant:

```rust
if let Ok([a, b, c]) = query.get_many_mut([entity_a, entity_b, entity_c]) {
    // Safe access
}
```

Immutable variants exist too: `many()` and `get_many()`.

### Iterating Over Entity Lists

When you have a dynamic list of entities, use `iter_many`:

```rust
#[derive(Component)]
struct Team {
    members: Vec<Entity>,
}

fn announce_team(
    blue_team: Query<&Team>,
    players: Query<&Player>
) {
    for team in &blue_team {
        println!("Team roster:");
        for player in players.iter_many(&team.members) {
            println!("  - {}", player.name);
        }
    }
}
```

For mutable iteration over entity lists, use `iter_many_mut`. Since it must prevent aliased mutability, it doesn't implement `Iterator`:

```rust
fn update_team_scores(
    blue_team: Query<&Team>,
    mut players: Query<&mut Player>
) {
    for team in &blue_team {
        let mut iter = players.iter_many_mut(&team.members);
        while let Some(mut player) = iter.fetch_next() {
            player.score += 10;
        }
    }
}
```

The `fetch_next()` pattern ensures safe mutable access even when iterating over arbitrary entity lists.

### Converting Queries to Read-Only

Mutable queries can be converted to read-only versions, useful for building abstractions:

```rust
fn update_and_log(mut players: Query<&mut Player>) {
    // Mutate players
    for mut player in &mut players {
        player.score += 1;
    }
    
    // Convert to read-only for logging
    log_players(players.to_readonly());
}

fn log_players(players: Query<&Player>) {
    for player in &players {
        println!("Player {} has {} points", player.name, player.score);
    }
}
```

This enables passing mutable queries to functions expecting read-only queries, making code more reusable. Once converted to read-only, you can't convert back - the change is one-way.

### Conflicting Queries

If you need overlapping mutable access, use ParamSet:

```rust
fn system(mut queries: ParamSet<(
    Query<&mut Health>,
    Query<(&mut Health, &Armor)>
)>) {
    // First query
    for health in queries.p0().iter() {
        println!("Health: {}", health.current);
    }
    
    // Second query (overlaps with first)
    for (mut health, armor) in queries.p1().iter_mut() {
        health.current += armor.regeneration;
    }
}
```

ParamSet ensures only one query is active at a time, preventing unsafe mutable aliasing while still allowing both queries in the same system.

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
    let entity = commands.spawn(EnemyBundle::default()).id();
    
    // Spawn and add components incrementally
    commands.spawn((Position::default(), Velocity::default()))
        .insert(Enemy);
}
```

### Modifying Entities

```rust
// Example using user-defined marker components
#[derive(Component)]
struct Defeated;

#[derive(Component)]
struct Despawning;

fn mark_defeated(mut commands: Commands, query: Query<Entity, With<Defeated>>) {
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

Entity hierarchies use two components: `Parent` (points to parent entity) and `Children` (points to child entities). This separation makes it easy to query hierarchy roots:

```rust
fn find_roots(roots: Query<Entity, Without<Parent>>) {
    // Entities without parents are hierarchy roots
}
```

Hierarchy changes must be done transactionally through Commands to maintain consistency. Direct modification of `Parent` and `Children` components is not allowed - this ensures the hierarchy is always correct.

Build hierarchies with `with_children`:

```rust
commands
    .spawn(SpatialBundle::default())
    .with_children(|parent| {
        parent.spawn(SpriteBundle {
            texture: player_texture,
            ..Default::default()
        });
        parent.spawn(SpriteBundle {
            texture: hat_texture,
            ..Default::default()
        });
    });
```

Manipulate existing hierarchies:

```rust
// Add children to an existing entity
commands.entity(parent)
    .with_children(|parent| {
        parent.spawn(ChildBundle::default());
    });

// Add specific entity as child
commands.entity(parent).add_child(child_entity);

// Remove children
commands.entity(parent).remove_children(&[child1, child2]);

// Despawn entity and all descendants
commands.entity(parent).despawn_descendants();

// Despawn entity, keep children (orphan them)
commands.entity(parent).despawn();
```

The transactional approach guarantees the hierarchy is never in an invalid state. All hierarchy changes happen atomically when commands are applied.

### Hierarchy Query Methods

Navigate hierarchies easily with convenience methods on `Query<&Children>` and `Query<&Parent>`:

```rust
#[derive(Resource)]
struct PlayerEntity(Entity);

fn check_descendants(
    children: Query<&Children>,
    player: Res<PlayerEntity>
) {
    // Iterate all descendants (children, grandchildren, etc.)
    for entity in children.iter_descendants(player.0) {
        println!("Found descendant: {:?}", entity);
    }
}

fn check_ancestors(
    parents: Query<&Parent>,
    player: Res<PlayerEntity>
) {
    // Iterate all ancestors (parent, grandparent, etc.)
    for entity in parents.iter_ancestors(player.0) {
        println!("Found ancestor: {:?}", entity);
    }
}
```

These methods traverse the entire hierarchy tree automatically. `iter_descendants` performs a depth-first search through all children, grandchildren, and so on. `iter_ancestors` walks up the hierarchy from the entity to the root.

Use these for:
- Finding all entities beneath a parent (useful for batch operations)
- Checking if an entity is part of a specific hierarchy branch
- Propagating changes through a hierarchy
- Collecting all related entities for processing

### Resource Initialization

Initialize resources with their default values:

```rust
fn setup(mut commands: Commands) {
    // Initialize a resource using Default
    commands.init_resource::<GameConfig>();
    
    // Equivalent to:
    // commands.insert_resource(GameConfig::default());
}
```

This is convenient when you just need a resource initialized to its default state without constructing it explicitly.

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
- `&World` - Read-only access to the entire world

### Direct World Access

Systems can access the entire World directly:

```rust
fn debug_system(world: &World) {
    println!("Entity count: {}", world.entities().len());
    
    if let Some(time) = world.get_resource::<Time>() {
        println!("Time: {:.2}s", time.elapsed_seconds());
    }
}
```

Note that `&World` conflicts with mutable queries. Use ParamSet to resolve:

```rust
fn system(mut set: ParamSet<(&World, Query<&mut Transform>)>) {
    let entity_count = set.p0().entities().len();
    
    for mut transform in set.p1().iter_mut() {
        transform.translation.x += 1.0;
    }
}
```

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

## System Ordering

Bevy runs systems in parallel when possible. Sometimes you need explicit ordering - one system must run before or after another:

```rust
fn update_velocity(mut query: Query<&mut Velocity>) {
    // Update velocities based on input or AI
}

fn apply_velocity(mut query: Query<(&Velocity, &mut Transform)>) {
    for (velocity, mut transform) in query.iter_mut() {
        transform.translation += velocity.0;
    }
}

fn main() {
    App::new()
        .add_system(update_velocity)
        .add_system(apply_velocity.after(update_velocity))
        .run();
}
```

The `.after()` method ensures `apply_velocity` runs after `update_velocity` finishes. Use `.before()` for the opposite:

```rust
.add_system(physics.before(apply_velocity))
```

### System Labels

For complex ordering, use explicit labels:

```rust
#[derive(SystemLabel)]
enum GameSystem {
    Input,
    Logic,
    Movement,
    Rendering,
}

App::new()
    .add_system(handle_input.label(GameSystem::Input))
    .add_system(game_logic.label(GameSystem::Logic).after(GameSystem::Input))
    .add_system(movement.label(GameSystem::Movement).after(GameSystem::Logic))
    .add_system(render.label(GameSystem::Rendering).after(GameSystem::Movement))
    .run();
```

Labels let multiple systems share the same ordering point. Any system labeled with `GameSystem::Logic` runs after all `GameSystem::Input` systems.

### Ordering vs Parallelism

Bevy parallelizes systems aggressively. Only add ordering when necessary - unnecessary constraints limit parallelism. Systems with no conflicts run in parallel automatically.

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

