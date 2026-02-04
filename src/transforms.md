# Transforms

Transforms control the position, rotation, and scale of entities in Bevy.

## Transform System Rewrite in Bevy 0.2

**Changed in Bevy 0.2**

The transform system underwent a major rewrite between Bevy 0.1 and 0.2.

### Old System (Bevy 0.1)

The old transform system used separate Translation, Rotation, and Scale components as the "source of truth":

```rust
// Bevy 0.1
fn system(translation: &Translation, rotation: &Rotation, scale: &Scale) {
    println!("{} {} {}", translation.0, rotation.0, scale.0);
}
```

Users modified these components in their systems, after which they were synced to a LocalTransform component, which was in turn synced to a global Transform component, taking hierarchy into account.

**Benefits:**
- Slightly more cache efficient to retrieve individual components (less data needs to be accessed)
- Theoretically more parallel-friendly - systems that only access Translation won't block systems accessing Rotation

**Downsides:**
- The individual components are the source of truth, so LocalTransform is out of date when user systems are running. If an up-to-date "full transform" is needed, it must be manually constructed by accessing all three components.
- Very hard to reason about - 5 components users need to think about and they all interact with each other differently
- Setting a Transform to a specific matrix value (ex: `Mat4::look_at()`) was extremely cumbersome, and the value would be immediately overwritten unless the user explicitly disabled component syncing

### New System (Bevy 0.2)

The new system uses a single unified local-to-parent Transform component as the source of truth, and a computed GlobalTransform component for world-space transforms:

```rust
// Bevy 0.2
fn system(transform: &Transform) {
    println!("{} {} {}", 
        transform.translation(), 
        transform.rotation(), 
        transform.scale()
    );
}
```

**Benefits:**
- Much easier to use and reason about
- Up-to-date transform always available
- Direct matrix manipulation is straightforward
- Only 2 components to think about

## Working with Transforms

### Accessing Transform Properties

```rust
fn system(transform: &Transform) {
    let translation = transform.translation();
    let rotation = transform.rotation();
    let scale = transform.scale();
}
```

### Modifying Transforms

```rust
fn system(mut transform: Mut<Transform>) {
    // Transform modification here
}
```

### Global Transforms

The `GlobalTransform` component contains the world-space transform, taking parent transforms into account. It is automatically computed by Bevy.

```rust
fn system(global_transform: &GlobalTransform) {
    // Read world-space transform
}
```

## Optimization

**Added in Bevy 0.2**

Transform systems are optimized to only run on changes, avoiding unnecessary recalculation when transforms haven't been modified.

