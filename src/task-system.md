# Task System

Bevy uses an efficient task system for multi-threading across the engine: ECS parallel systems, asset loading, rendering, and background work all run on thread pools.

## Parallel Execution

Modern CPUs have multiple cores. Games should use them all. Bevy's task system makes parallel execution straightforward while managing resources efficiently.

Single-threaded code leaves performance on the table. A game running on one core while seven others sit idle wastes potential. But naive multithreading creates its own problems: contention, synchronization overhead, and excessive CPU usage even when idle.

## Task Pools

Bevy provides specialized thread pools for different work types:

**ComputeTaskPool** - CPU-intensive work like physics calculations or procedural generation.

**AsyncComputeTaskPool** - Async computation that may wait on external events.

**IOTaskPool** - File operations, network requests, and other I/O.

Separating work by type prevents I/O operations from blocking compute tasks and allows different scheduling strategies for different workload types.

## Parallel Iteration

Distribute query iteration across threads:

```rust
// Example with user-defined marker component
#[derive(Component)]
struct Moving;

fn parallel_movement(
    pool: Res<ComputeTaskPool>,
    mut query: Query<&mut Transform, With<Moving>>
) {
    query.par_iter_mut().for_each(&pool, |mut transform| {
        transform.translation.y -= 9.8 * 0.016; // Gravity
    });
}
```

This splits the query into batches, processing each batch on a different thread. The work happens in parallel automatically.

## Spawning Tasks

Execute arbitrary work on task pools:

```rust
fn expensive_computation(pool: Res<ComputeTaskPool>) {
    let task = pool.spawn(async move {
        // This runs on a background thread
        perform_expensive_calculation()
    });
    
    // Continue with other work
    // ...
    
    // Wait for result when needed
    let result = futures_lite::future::block_on(task);
}
```

Tasks return futures that complete when the work finishes. This enables fire-and-forget operations or explicit synchronization when needed.

## Scope for Borrowed Data

Sometimes tasks need to access data that outlives the task:

```rust
fn process_data(
    pool: Res<ComputeTaskPool>,
    data: &[f32]
) {
    pool.scope(|scope| {
        for chunk in data.chunks(1000) {
            scope.spawn(async move {
                process_chunk(chunk);
            });
        }
    });
    // All spawned tasks complete before scope returns
}
```

Scopes ensure all spawned tasks finish before continuing, making it safe to access borrowed data from multiple tasks.

### Nested Spawns

Tasks can spawn additional tasks within a scope:

```rust
fn nested_tasks(pool: Res<ComputeTaskPool>) {
    let results = pool.scope(|scope| {
        scope.spawn(async move {
            // Spawn another task from within this task
            scope.spawn(async move { 1 });
            2
        });
    });
    
    assert!(results.contains(&1));
    assert!(results.contains(&2));
}
```

Nested spawning enables dynamic task creation based on intermediate results. Tasks can branch into subtasks as complexity unfolds, adapting the work structure at runtime.

### Panic Handling

Task pools now catch panics in worker tasks, preventing a single panic from killing worker threads:

```rust
fn safe_task_execution(pool: Res<ComputeTaskPool>) {
    pool.scope(|scope| {
        scope.spawn(async {
            // If this panics, it won't crash the entire pool
            risky_operation();
        });
        
        scope.spawn(async {
            // This task still runs even if the previous one panicked
            safe_operation();
        });
    });
}
```

Previously, a panicking task would permanently disable that worker thread. Now panics are contained, logged, and don't affect other tasks or threads. The task pool remains healthy and continues processing work.

## Efficient Resource Usage

Bevy's task system uses resources proportionally to actual work. Idle threads sleep rather than spinning, reducing CPU usage and power consumption.

This contrasts with aggressive work-stealing schedulers that maximize throughput at the cost of constant CPU activity. Bevy balances performance with efficiency - threads do real work or rest, not busy-waiting.

## Async Integration

The task system integrates with Rust's async ecosystem:

```rust
async fn load_assets(server: &AssetServer) -> Vec<Handle<Mesh>> {
    let handles = vec![
        server.load("model1.gltf"),
        server.load("model2.gltf"),
        server.load("model3.gltf"),
    ];
    
    // Can use async/.await syntax
    handles
}
```

This enables asynchronous patterns without blocking threads, crucial for I/O-heavy operations.

## When to Use Parallel Execution

**Use parallel execution when:**
- Processing large collections of entities
- Performing expensive per-entity calculations
- Operating on independent data
- Work can be split into uniform chunks

**Don't use parallel execution when:**
- Collections are small (overhead exceeds benefit)
- Work requires sequential ordering
- Tasks need frequent synchronization
- Simple operations complete faster sequentially

Measure before parallelizing. Sometimes the simplest solution is fastest.

## Best Practices

**Batch appropriately** - Too many tiny tasks create overhead. Too few large tasks underutilize cores. Aim for hundreds to thousands of operations per batch.

**Avoid contention** - Parallel tasks shouldn't write to shared state. Use thread-local data or message passing instead.

**Use the right pool** - Compute work on ComputeTaskPool, I/O on IOTaskPool. Don't mix workload types.

**Profile** - Measure actual performance. Parallel code doesn't automatically run faster.

**Keep tasks independent** - Tasks that need to synchronize frequently should probably run sequentially.

Bevy's task system provides the tools for efficient parallel execution. Understanding when and how to use them maximizes performance without wasting resources.

