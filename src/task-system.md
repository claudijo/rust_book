# Task System

**Added in Bevy 0.2**

Bevy uses a custom async-friendly task system for multi-threading throughout the engine: ECS scheduling, asset loading, rendering, and more.

## Why a Custom Task System?

Before Bevy 0.2, the engine used Rayon for most multi-threading tasks. Rayon is nice because it is generally as simple as calling `some_list.par_iter().for_each(|x| do_something(x))`. Rayon then automatically breaks the for_each into tasks and runs them on as many cores as it can. While Rayon is a great choice for easily parallelizing code, it has the downside of being pretty CPU-hungry.

Bevy (and other Rust game engines and ECS frameworks using Rayon) received feedback that they were overly CPU hungry - usage was not proportional to "real" work done.

## The Solution

Bevy 0.2 introduced a custom async-friendly task system that enables the creation of context-specific task pools. For example, you might have separate pools for compute, IO, networking, etc. This also gives the flexibility to load balance work appropriately according to work type and/or priority.

## CPU Usage Improvements

The CPU usage wins have been huge:

**8 Core Machine** - Total Combined Percent CPU Usage (smaller is better):
- Bevy 0.1 (Rayon): High CPU usage
- Bevy 0.2 (Task System): Significantly reduced CPU usage

**32 Core Machine** - Total Combined Percent CPU Usage (smaller is better):
- The improvements are even more dramatic on machines with more cores

## Task Pools

Bevy exports several task pools in the `bevy_tasks` crate:

- **IOTaskPool** - For I/O operations
- **ComputeTaskPool** - For compute-heavy operations  
- **AsyncComputeTaskPool** - For async compute operations

## Usage

Task pools can be accessed as resources in systems:

```rust
fn system(pool: Res<ComputeTaskPool>, mut query: Query<&mut Transform>) {
    query.iter().par_iter(32).for_each(&pool, |mut transform| {
        transform.translate(Vec3::new(1.0, 0.0, 0.0));
    });
}
```

This distributes work across the compute task pool, enabling efficient parallel processing while keeping CPU usage proportional to the actual work being done.

## Benefits

- **Lower CPU usage** - More efficient than Rayon for game engine workloads
- **Context-specific pools** - Different work types can use different pools
- **Load balancing** - Work can be balanced according to type and priority
- **Async-friendly** - Integrates well with Rust's async ecosystem

