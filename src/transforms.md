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

### Directional Vectors

**Added in Bevy 0.6**

Get direction vectors relative to the transform:

```rust
fn system(transform: &Transform) {
    let left = transform.left();      // Points to the left
    let right = transform.right();    // Points to the right
    let up = transform.up();          // Points up
    let down = transform.down();      // Points down
    let forward = transform.forward(); // Points forward
    let back = transform.back();      // Points back
}
```

These return `Vec3` vectors in world space, useful for movement and orientation calculations:

```rust
fn movement_system(
    input: Res<Input<KeyCode>>,
    mut query: Query<&mut Transform, With<Player>>
) {
    for mut transform in query.iter_mut() {
        if input.pressed(KeyCode::W) {
            transform.translation += transform.forward() * 0.1;
        }
        if input.pressed(KeyCode::A) {
            transform.translation += transform.left() * 0.1;
        }
    }
}
```

### Builder Methods

**Added in Bevy 0.6**

Chain transform creation with builder methods:

```rust
let transform = Transform::from_xyz(0.0, 0.0, 10.0)
    .with_scale(Vec3::splat(2.0))
    .with_rotation(Quat::from_rotation_y(PI / 4.0));

// Available methods:
// - with_translation(Vec3)
// - with_rotation(Quat)
// - with_scale(Vec3)
```

Clean and expressive!

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

