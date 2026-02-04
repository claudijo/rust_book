# Bevy ECS

All Bevy engine and game logic is built on top of a custom Entity Component System (ECS).

## What is ECS?

Entity Component Systems are a software paradigm that involves breaking data up into **Components**. **Entities** are unique ids assigned to groups of Components. For example, one entity might have a Position and Velocity component, whereas another entity might have a Position and UI component. **Systems** are logic that runs on a specific set of component types. You might have a movement system that runs on all entities with a Position and Velocity component.

The ECS pattern encourages clean, decoupled designs by forcing you to break up your app data and logic into its core components.

## Bevy's Approach

Unlike other Rust ECS implementations, which require complex lifetimes, traits, builder patterns, or macros, Bevy ECS uses normal Rust datatypes for all of these concepts:

- **Components**: normal Rust structs
- **Systems**: normal Rust functions
- **Entities**: a type containing a unique integer

## Ergonomics

Bevy ECS is designed to be the most ergonomic ECS in existence. Here's a complete example:

```rust
use bevy::prelude::*;

struct Velocity(f32);
struct Position(f32);

// this system spawns entities with the Position and Velocity components
fn setup(mut commands: Commands) {
    commands
        .spawn((Position(0.0), Velocity(1.0),))
        .spawn((Position(1.0), Velocity(2.0),));
}

// this system runs on each entity with a Position and Velocity component
fn movement(mut position: Mut<Position>, velocity: &Velocity) {
    position.0 += velocity.0;
}

// the app entry point
fn main() {
    App::build()
        .add_default_plugins()
        .add_startup_system(setup.system())
        .add_system(movement.system())
        .run();
}
```

That is a complete self-contained Bevy app with automatic parallel system scheduling and global change detection.

## Performance

The ECS paradigm has the potential to make game logic super fast for two main reasons:

1. **Iteration Speed**: Components are packed tightly together to optimize for cache-locality, which makes iterating over them blazing fast
2. **Parallelism**: Systems declare read/write dependencies, which enables automatic and efficient lock-free parallel scheduling

Bevy ECS does both of these things about as well as it can. According to the popular ecs_bench benchmark, Bevy ECS is the fastest Rust ECS by a pretty wide margin in both system iteration and world setup.

> Note: ecs_bench is a single threaded benchmark, so it doesn't illustrate the multi-threading capabilities. As always, be aware that ecs_bench is a micro benchmark and doesn't illustrate the performance of a complex game.

## System Types

### For Each Systems

"For each systems" run once on each entity containing the given components:

```rust
fn system(position: Mut<Position>, velocity: &Velocity) {
    // do something
}
```

### Query Systems

Query systems give you control over iteration:

```rust
fn system(mut query: Query<(&Position, &mut Velocity)>) {
    for (position, mut velocity) in &mut query.iter() {
        // do something
    }
}
```

### Change Detection

**Added** queries only run when the given component has been added:

```rust
fn system(mut query: Query<Added<Position>>) {
    for position in &mut query.iter() {
        // do something
    }
}
```

**Mutated** queries only run when the given component has been mutated:

```rust
fn system(mut query: Query<Mutated<Position>>) {
    for position in &mut query.iter() {
        // do something
    }
}
```

**Changed** queries only run when the given component has been added or mutated:

```rust
fn system(mut query: Query<Changed<Position>>) {
    for position in &mut query.iter() {
        // do something
    }
}
```

**Removed** iterates over every entity where the component was removed this update:

```rust
fn system(mut query: Query<&Position>) {
    for entity in query.removed::<Velocity>() {
        // do something
    }
}
```

### Multiple Queries

Systems can have multiple queries:

```rust
fn system(mut wall_query: Query<&Wall>, mut player_query: Query<&Player>) {
    for player in &mut player_query.iter() {
        for wall in &mut wall_query.iter() {
            if player.collides_with(wall) {
                println!("ouch");
            }
        }
    }
}
```

### Parallel Queries

**Added in Bevy 0.2**

Queries can now be iterated in parallel to distribute work across multiple threads:

```rust
fn system(pool: Res<ComputeTaskPool>, mut query: Query<&mut Transform>) {
    query.iter().par_iter(32).for_each(&pool, |mut transform| {
      transform.translate(Vec3::new(1.0, 0.0, 0.0));
    });
}
```

This breaks the query up into 32 "batches" and runs each batch as a different task in the bevy task system.

### Or Queries

**Added in Bevy 0.2**

Support for Or in ECS queries allows matching entities that have any of the specified components:

```rust
// Matches entities with either ComponentA or ComponentB
fn system(query: Query<Or<&ComponentA, &ComponentB>>) {
    // ...
}
```

### Entity Queries and Direct Component Access

```rust
fn system(mut entity_query: Query<Entity>, mut player_query: Query<&Player>) {
    for entity in &mut entity_query.iter() {
       if let Some(player) = player_query.get::<Player>(entity) {
           // the current entity has a player component
       }
    }
}
```

## Resources

`Res` and `ResMut` access global resources:

```rust
fn system(time: Res<Time>, score: ResMut<Score>) {
    // do something
}
```

You can use Resources in any system type:

```rust
fn system(time: Res<Time>, mut query: Query<&Position>) {
    // do something
}

fn system(time: Res<Time>, &Position) {
    // do something
}
```

### Local System Resources

`Local<T>` resources are unique per-system. Two instances of the same system will each have their own resource. Local resources are automatically initialized to their default value:

```rust
fn system(state: Local<State>, &Position) {
    // do something
}
```

## Other System Features

### Empty Systems

For the hyper-minimalists:

```rust
fn system() {
    // do something
}
```

### With/Without Filters

Only runs on entities With or Without a given component:

```rust
fn system(mut query: Query<Without<Parent, &Position>>) {
    for position in &mut query.iter() {
        // do something
    }
}
```

### Thread-Local Systems

Systems that must run on the main thread with exclusive access to World and Resources:

```rust
fn system(world: &mut World, resources: &mut Resources) {
    // do something
}
```

## Stages

The scheduler provides Stages as a way to run sets of systems in order:

```rust
fn main() {
    App::build()
        // adds a system to the default stage: "update"
        .add_system(movement.system())
        // creates a new stage after "update"
        .add_stage_after("update", "do_things")
        .add_system_to_stage("do_things", something.system())
}
```

## Commands

Use Commands to queue up World and Resource changes, which will be applied at the end of the current Stage:

```rust
fn system(mut commands: Commands) {
    commands.spawn((Position(0.0), Velocity(1.0)));
}
```

Commands can also be used alongside other types:

```rust
fn system(mut commands: Commands, time: Res<Time>, mut query: Query<&Position>) {
    // do something
}
```

## How Function Systems Work

Being able to use Rust functions directly as systems might feel like magic, but it's not! When registering systems in your App:

```rust
fn some_system() { }

fn main() {
    App::build()
        .add_system(some_system.system())
        .run();
}
```

The `.system()` call takes the function pointer and converts it to a `Box<dyn System>`. This works because Bevy implements the `IntoQuerySystem` trait for all functions that match a certain set of function signatures.

## Performance Improvements in Bevy 0.2

**Added in Bevy 0.2**

Several significant performance improvements were made to Bevy ECS:

### Generational Entity IDs

Entity IDs changed from random UUIDs to incrementing generational indices. Random UUIDs were nice because they could be created anywhere, were unique across game runs, and could be safely persisted to files or reused across networks. However, they ended up being too slow relative to the alternatives. The randomness had a measurable cost and entity locations had to be looked up using a hash map.

By moving to generational indices (using the hecs implementation), entity ids can be directly used as array indices, which makes entity location lookups lightning fast.

### Read-Only Queries

Read-only traits were implemented for queries that don't mutate anything. This allows guaranteeing that a query won't mutate anything, enabling further optimizations.

### Removed Locking from World APIs

This gives a significant speed boost. This can be done safely due to a combination of the new read-only queries and changing World mutation APIs to be a mutable World borrow.

As a result of these optimizations, direct component lookup is much faster.

## Built on Hecs

Bevy ECS uses a heavily forked version of the minimalist Hecs ECS. Hecs is an efficient single-threaded archetypal ECS that provides the core World, Archetype, and internal Query data structures. Bevy ECS adds on top:

- **Function Systems**: Hecs has no concept of a "system" at all - you just run queries directly on the World. Bevy adds portable, schedulable systems using normal Rust functions.
- **Resources**: Hecs has no concept of unique/global data. Bevy adds a Resource collection and resource queries.
- **Parallel Scheduler**: Hecs is single threaded. Bevy ECS adds a custom dependency-aware scheduler.
- **Optimization**: By modifying internal data access patterns, Bevy improved performance significantly.
- **Query Wrappers**: Bevy's Query provides safe, scoped access to the World in a multi-threaded context with improved ergonomics.
- **Change Detection**: Automatically and efficiently tracks component add/remove/update operations.
- **Entity IDs**: In Bevy 0.1, entity ids were globally unique UUIDs suitable for serialization. In Bevy 0.2, this changed to generational indices for better performance, using direct array indexing for lookups.

