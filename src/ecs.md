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

**Deprecated in Bevy 0.4**

"For each systems" were available in Bevy 0.1-0.3 and ran once for each entity:

```rust
// Bevy 0.1-0.3 only (deprecated in 0.4)
fn system(position: Mut<Position>, velocity: &Velocity) {
    // ran once per entity
}
```

**Why were they removed?**
1. Fundamentally limited - couldn't filter, use multiple queries, or iterate removed components
2. Forced design decisions - "one way to do things" is better
3. Common "gotchas" for newcomers (e.g., `&mut T` vs `Mut<T>` confusion)
4. Increased compile times (~5 seconds saved by removing)
5. Complex macro implementation

**Use Query Systems Instead (Bevy 0.4+):**
```rust
fn system(query: Query<(&mut Position, &Velocity)>) {
    for (mut position, velocity) in query.iter_mut() {
        // per-entity logic here
    }
}
```

### Query Systems

Query systems give you control over iteration:

**Bevy 0.1-0.2:**
```rust
fn system(mut query: Query<(&Position, &mut Velocity)>) {
    for (position, mut velocity) in &mut query.iter() {
        // do something
    }
}
```

**Bevy 0.3+ (improved ergonomics):**
```rust
fn system(mut query: Query<(&Position, &mut Velocity)>) {
    // query.iter() is now a real iterator! No more &mut required
    for (position, mut velocity) in query.iter() {
        // sweet ergonomic bliss
    }
}
```

The 0.3 improvement removed the awkward `&mut` from iteration and made `query.iter()` return an actual iterator instead of a wrapper type.

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

### QuerySets and 100% Lockless ECS

**Added in Bevy 0.3**

Bevy ECS became completely lock-free in version 0.3. Previously, locks were needed to prevent unsafe access when systems had conflicting queries like this:

```rust
// This would have been unsafe without locks
fn conflicting_query_system(mut q0: Query<&mut A>, mut q1: Query<(&mut A, &B)>) {
    let a = q0.get_mut(some_entity).unwrap();
    let (another_a, b) = q1.get_mut(some_entity).unwrap();
    // Aaah!!! Two mutable references to the same component!
}
```

In Bevy 0.3, systems with conflicting queries fail when the schedule is constructed. For cases where conflicting queries are needed, **QuerySets** were added:

```rust
fn system(mut queries: QuerySet<(Query<&mut A>, Query<(&mut A, &B)>)>) {
    for a in queries.q0_mut().iter_mut() {
        // Access first query
    }

    for (a, b) in queries.q1_mut().iter_mut() {
        // Access second query  
    }
}
```

By putting conflicting Queries in a QuerySet, the Rust borrow checker protects us from unsafe query accesses. This allowed removal of all safety checks and locks from query access, making Bevy ECS 100% lock-free.

### Query API Changes in Bevy 0.3

**Changed in Bevy 0.3**

Several query methods were renamed and improved:

- `query.entity(entity)` → `query.get(entity)` (returns full query result)
- `query.get::<Component>(entity)` → `query.get_component::<Component>(entity)` (for single component access)
- Added separate `query.iter()` and `query.iter_mut()` for read-only and mutable iteration
- Added `query.get(entity)` and `query.get_mut(entity)` for entity-specific access

**Before (0.2):**
```rust
if let Ok(mut result) = query.entity(entity) {
    if let Some((a, b)) = result.get() {
        // access components here
    }
}
```

**After (0.3):**
```rust
if let Ok((a, b)) = query.get(entity) {
    // boilerplate be gone!
}
```

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

### Thread Local Resources

**Added in Bevy 0.3**

Some resource types cannot (or should not) be passed between threads. This is often true for low-level APIs like windowing, input, and audio. Thread local resources can only be accessed from the main thread using "thread local systems":

```rust
// In your app setup
app.add_thread_local_resource(MyResource);

// A thread local system
fn system(world: &mut World, resources: &mut Resources) {
    let my_resource = resources.get_thread_local::<MyResource>().unwrap();
    // do something with my_resource
}
```

Thread local systems have exclusive access to the World and Resources, and must run on the main thread.

## Other System Features

### Empty Systems

For the hyper-minimalists:

```rust
fn system() {
    // do something
}
```

### With/Without Filters

**Bevy 0.1-0.3 (confusing nested syntax):**
```rust
// Filters were intermingled with components
fn system(query: Query<With<A, Without<B, (&Transform, Changed<Velocity>)>>>) {
    // Hard to tell what's a filter vs what's a component!
}
```

**Bevy 0.4+ (separated filters):**
```rust
// Query filters are now separate from components
fn system(query: Query<(&Transform, &Velocity), (With<A>, Without<B>, Changed<Velocity>)>) {
    // Much clearer! Second tuple is filters only
}

// Query without filters
fn system(query: Query<(&Transform, &Velocity)>) {
    // Clean and simple
}
```

Benefits of the new syntax:
- Clearer what the query returns vs what it filters on
- Can filter on `Changed<Velocity>` without retrieving Velocity
- Can create type aliases for reusable filters:

```rust
type MovingEntities = (With<A>, Without<B>, Changed<Velocity>);

fn system(query: Query<(&Transform, &Velocity), MovingEntities>) {
    // Reusable filter!
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

**Bevy 0.4 Changes:**

Commands changed from `mut commands: Commands` to `commands: &mut Commands`:

```rust
fn system(commands: &mut Commands, time: Res<Time>) {
    commands.spawn((Position(0.0), Velocity(1.0)));
}
```

## Flexible System Parameters

**Added in Bevy 0.4**

Prior to 0.4, system parameters had to be in a specific order ([Commands][Resources][Queries]), which was confusing and error-prone.

**Bevy 0.1-0.3 (required order):**
```rust
// This compiled
fn valid(commands: &mut Commands, time: Res<Time>, query: Query<&Transform>) {}

// This failed to compile!
fn invalid(query: Query<&Transform>, commands: &mut Commands, time: Res<Time>) {}
```

**Bevy 0.4+ (any order works!):**
```rust
// All of these work now!
fn system1(query: Query<&Transform>, commands: &mut Commands, time: Res<Time>) {}
fn system2(time: Res<Time>, query: Query<&Transform>, commands: &mut Commands) {}
fn system3(commands: &mut Commands, query: Query<&Transform>, time: Res<Time>) {}
```

This was achieved by completely rewriting system generation using a new `SystemParam` trait. Benefits:
- **~25% faster clean compile times**
- **Any parameter order works**
- **Easy to add new parameters** - just implement SystemParam
- **Simpler implementation** - much easier to maintain

## System Inputs, Outputs, and Chaining

**Added in Bevy 0.4**

Systems can now have inputs and outputs, enabling powerful patterns like error handling:

```rust
fn main() {
    App::build()
        .add_system(result_system.system().chain(error_handler.system()))
        .run();
}

fn result_system(query: Query<&Transform>) -> Result<(), MyError> {
    let transform = query.get(SOME_ENTITY)?;
    println!("found transform: {:?}", transform);
    Ok(())
}

fn error_handler(In(result): In<Result<(), MyError>>) {
    if let Err(err) = result {
        println!("Error: {:?}", err);
    }
}
```

The System trait signature:
```rust
System<In = (), Out = ()>        // No inputs or outputs
System<In = usize, Out = f32>    // Takes usize, returns f32
```

This enables:
- Error handling chains
- Data pipelines between systems
- Composable system logic

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

## Performance Improvements in Bevy 0.3

**Added in Bevy 0.3**

Bevy 0.3 brought additional significant performance improvements:

### Removed All Locks from Query Access

By introducing QuerySets for conflicting queries, all atomic locks could be removed from query access. This makes queries exactly as fast as direct World access.

### Removed Archetype Safety Checks

Since queries are verified once during schedule construction, safety checks don't need to happen on every query access. This reduced overhead significantly.

### Rewritten QueryIter

QueryIter was completely rewritten to be simpler and more optimizable. This resolved performance inconsistencies where some system patterns performed better than others. Now all query iteration is on the "fast path".

### Upstream hecs Improvements

Ported performance improvements from upstream hecs:
- Improved iteration over heavily fragmented archetypes
- Improved component insertion times

### Benchmark Results

**Entity Component Lookup** (per 100k operations, smaller is better):
- Bevy 0.2: ~40-50ms with locks
- Bevy 0.3: ~5-10ms without locks (5-10x faster!)

**Component Insertion**: Improved significantly
**Fragmented Iteration**: Much faster for complex archetype structures

The query access improvements make Bevy ECS queries as fast as direct World access while maintaining safety through compile-time checks.

## Built on Hecs

Bevy ECS uses a heavily forked version of the minimalist Hecs ECS. Hecs is an efficient single-threaded archetypal ECS that provides the core World, Archetype, and internal Query data structures. Bevy ECS adds on top:

- **Function Systems**: Hecs has no concept of a "system" at all - you just run queries directly on the World. Bevy adds portable, schedulable systems using normal Rust functions.
- **Resources**: Hecs has no concept of unique/global data. Bevy adds a Resource collection and resource queries.
- **Parallel Scheduler**: Hecs is single threaded. Bevy ECS adds a custom dependency-aware scheduler.
- **Optimization**: By modifying internal data access patterns, Bevy improved performance significantly.
- **Query Wrappers**: Bevy's Query provides safe, scoped access to the World in a multi-threaded context with improved ergonomics.
- **Change Detection**: Automatically and efficiently tracks component add/remove/update operations.
- **Entity IDs**: In Bevy 0.1, entity ids were globally unique UUIDs suitable for serialization. In Bevy 0.2, this changed to generational indices for better performance, using direct array indexing for lookups.

