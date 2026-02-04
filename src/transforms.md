# Transforms

Transforms control the position, rotation, and scale of entities in Bevy.

## Transform Evolution

The transform system has undergone multiple rewrites to find the optimal API and implementation.

## Bevy 0.1: Separate Components

**Bevy 0.1** used separate Translation, Rotation, and Scale components as the "source of truth":

```rust
// Bevy 0.1
fn system(translation: &Translation, rotation: &Rotation, scale: &Scale) {
    println!("{} {} {}", translation.0, rotation.0, scale.0);
}
```

Users modified these components, which were then synced to LocalTransform and Transform components.

**Benefits:**
- Slightly more cache efficient for individual component access
- More parallel-friendly (different systems could access different components)

**Downsides:**
- LocalTransform out of date during user systems
- Hard to reason about (5 components)
- Matrix manipulation extremely cumbersome

## Bevy 0.2: Unified Transform with Matrix

**Changed in Bevy 0.2**

Moved to a single unified Transform component (4x4 matrix) as the source of truth, plus GlobalTransform for world-space:

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
- Simpler API (2 components instead of 5)
- Always up-to-date
- Easier matrix manipulation

**Remaining Issues:**
- Matrix as source of truth accumulates error over time
- API still somewhat cumbersome (accessors needed)
- More expensive to store and compute in some cases

## Bevy 0.3: Similarity Transform

**Changed in Bevy 0.3**

Transform was rewritten again to use a "similarity" (translation + rotation + scale) as the source of truth:

```rust
// Bevy 0.3+
fn system(mut transform: Mut<Transform>) {
    // Direct field access!
    transform.translation += Vec3::new(1.0, 0.0, 0.0);
    
    // Rotate 180 degrees (pi) around the y-axis
    transform.rotation *= Quat::from_rotation_y(PI);
    
    // Scale 2x
    transform.scale *= 2.0;
}
```

**Benefits:**
- **No error accumulation** - Similarity doesn't accumulate floating-point errors like matrices do
- **Direct field access** - translation, rotation, and scale are exposed as public fields
- **Simpler API** - No need for accessor methods
- **More efficient** - Cheaper to store and compute hierarchies in many cases
- **More correct** - Maintains numerical stability

**Transform Fields:**
- `translation: Vec3` - Position in 3D space
- `rotation: Quat` - Rotation as a quaternion
- `scale: Vec3` - Scale along each axis

This is the best version yet - easier to use, more correct, and faster!

This is the best version yet - easier to use, more correct, and faster!

## Working with Transforms

### Accessing Transform Properties

```rust
fn system(transform: &Transform) {
    let translation = transform.translation;  // Direct field access
    let rotation = transform.rotation;
    let scale = transform.scale;
}
```

### Modifying Transforms

```rust
fn system(mut transform: Mut<Transform>) {
    // Move
    transform.translation.x += 1.0;
    
    // Rotate
    transform.rotation *= Quat::from_rotation_y(0.1);
    
    // Scale
    transform.scale *= 1.01;
}
```

### Global Transforms

The `GlobalTransform` component contains the world-space transform, taking parent transforms into account. It is automatically computed by Bevy.

```rust
fn system(global_transform: &GlobalTransform) {
    // Read world-space transform
    let world_position = global_transform.translation;
}
```

## Optimization


Transform systems are optimized to only run on changes, avoiding unnecessary recalculation when transforms haven't been modified.

