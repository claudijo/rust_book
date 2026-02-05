# Transforms

Every visible entity in a game needs to know where it is, how it's oriented, and how big it is. Bevy's transform system handles these spatial properties through a simple but powerful API.

## The Transform Component

At the heart of Bevy's spatial system is the `Transform` component. It holds three essential pieces of information:

```rust
pub struct Transform {
    pub translation: Vec3,  // Where the entity is
    pub rotation: Quat,     // How it's oriented  
    pub scale: Vec3,        // How big it is
}
```

These fields are public and directly accessible, making transforms straightforward to work with:

```rust
fn move_player(mut query: Query<&mut Transform, With<Player>>) {
    for mut transform in query.iter_mut() {
        transform.translation.x += 5.0;  // Move right
        transform.rotation *= Quat::from_rotation_y(0.1);  // Rotate
        transform.scale *= 1.01;  // Grow slightly
    }
}
```

## Working with Position

The translation field determines where an entity appears in space. For 2D games, you typically work with X and Y coordinates, leaving Z at zero:

```rust
fn spawn_sprite(mut commands: Commands) {
    commands.spawn(SpriteBundle {
        transform: Transform::from_xyz(100.0, 50.0, 0.0),
        ..Default::default()
    });
}
```

For 3D games, all three axes matter:

```rust
fn spawn_cube(mut commands: Commands) {
    commands.spawn(PbrBundle {
        transform: Transform::from_xyz(0.0, 2.5, -5.0),
        ..Default::default()
    });
}
```

## Rotation and Quaternions

Bevy represents rotations using quaternions rather than Euler angles. While quaternions may seem unfamiliar at first, they solve several problems that plague Euler angles: they avoid gimbal lock, interpolate smoothly between orientations, and compose naturally.

In practice, you rarely construct quaternions directly. Instead, use the convenience methods:

```rust
// Rotate around an axis
transform.rotation = Quat::from_rotation_y(std::f32::consts::PI / 2.0);

// Apply additional rotation
transform.rotation *= Quat::from_rotation_z(0.1);

// Create from Euler angles if needed
transform.rotation = Quat::from_euler(EulerRot::XYZ, 0.0, PI / 4.0, 0.0);
```

## Scaling

The scale field controls size along each axis. Uniform scaling uses the same value for all dimensions:

```rust
transform.scale = Vec3::splat(2.0);  // Double size in all directions
```

Non-uniform scaling stretches or squashes along specific axes:

```rust
transform.scale = Vec3::new(2.0, 1.0, 1.0);  // Wider but same height and depth
```

## Movement Patterns

Transforms shine when combined with directional vectors. Rather than manually calculating movement directions, use the transform's orientation:

```rust
fn player_movement(
    input: Res<Input<KeyCode>>,
    mut query: Query<&mut Transform, With<Player>>
) {
    for mut transform in query.iter_mut() {
        let speed = 5.0;
        
        if input.pressed(KeyCode::W) {
            transform.translation += transform.forward() * speed;
        }
        if input.pressed(KeyCode::S) {
            transform.translation += transform.back() * speed;
        }
        if input.pressed(KeyCode::A) {
            transform.translation += transform.left() * speed;
        }
        if input.pressed(KeyCode::D) {
            transform.translation += transform.right() * speed;
        }
    }
}
```

These directional methods (`forward()`, `back()`, `left()`, `right()`, `up()`, `down()`) return unit vectors pointing in the corresponding directions based on the transform's current rotation. This makes character controls feel natural regardless of orientation.

## Pointing At Targets

The `look_at` method rotates a transform to face a target position:

```rust
fn enemies_track_player(
    player_query: Query<&Transform, With<Player>>,
    mut enemy_query: Query<&mut Transform, (With<Enemy>, Without<Player>)>
) {
    if let Ok(player_transform) = player_query.get_single() {
        for mut enemy_transform in enemy_query.iter_mut() {
            enemy_transform.look_at(
                player_transform.translation,
                Vec3::Y  // "up" direction
            );
        }
    }
}
```

This is invaluable for turrets, cameras following targets, or any entity that needs to face something.

## Hierarchies and Parent-Child Relationships

When entities form parent-child relationships, transforms become relative. A child's `Transform` is interpreted relative to its parent's position and orientation:

```rust
// Parent at world origin
commands.spawn(Transform::from_xyz(0.0, 0.0, 0.0))
    .with_children(|parent| {
        // Child is 5 units to the right of the parent
        parent.spawn(Transform::from_xyz(5.0, 0.0, 0.0));
    });
```

If the parent moves, children automatically move with it. If the parent rotates, children orbit around it while maintaining their relative positions. This is perfect for attaching weapons to characters, organizing level geometry, or creating articulated structures.

## Local vs World Space

Bevy maintains two transform representations:

**Transform** - Your local, relative transform. This is what you modify in your systems.

**GlobalTransform** - The computed world-space transform after applying all parent transforms. Bevy maintains this automatically.

You should only ever read `GlobalTransform`, never write to it. The transform propagation system updates it each frame based on `Transform` values and hierarchy relationships:

```rust
fn print_world_position(query: Query<&GlobalTransform>) {
    for global_transform in query.iter() {
        let world_pos = global_transform.translation();
        println!("World position: {:?}", world_pos);
    }
}
```

Always modify `Transform`. Bevy handles the rest.

## Building Transforms

Several convenience constructors make transform creation ergonomic:

```rust
// Start at specific position
let t = Transform::from_xyz(10.0, 0.0, 5.0);

// Start with rotation
let t = Transform::from_rotation(Quat::from_rotation_y(PI));

// Start with scale
let t = Transform::from_scale(Vec3::splat(2.0));

// Chain multiple properties
let t = Transform::from_xyz(0.0, 10.0, 0.0)
    .with_scale(Vec3::splat(2.0))
    .with_rotation(Quat::from_rotation_y(PI / 4.0));
```

## Common Patterns

Transform manipulation shows up in countless gameplay scenarios. Here are some frequent patterns:

**Following a target** at a fixed distance:

```rust
fn follow_camera(
    target: Query<&Transform, With<Player>>,
    mut camera: Query<&mut Transform, (With<Camera>, Without<Player>)>
) {
    if let Ok(target_transform) = target.get_single() {
        for mut camera_transform in camera.iter_mut() {
            let desired_pos = target_transform.translation + Vec3::new(0.0, 5.0, 10.0);
            camera_transform.translation = desired_pos;
            camera_transform.look_at(target_transform.translation, Vec3::Y);
        }
    }
}
```

**Orbiting** around a point:

```rust
fn orbit(time: Res<Time>, mut query: Query<&mut Transform, With<Orbiter>>) {
    for mut transform in query.iter_mut() {
        let radius = 10.0;
        let speed = 2.0;
        let angle = time.elapsed_seconds() * speed;
        
        transform.translation.x = angle.cos() * radius;
        transform.translation.z = angle.sin() * radius;
    }
}
```

**Billboard sprites** that always face the camera:

```rust
fn billboard_sprites(
    camera: Query<&Transform, With<Camera>>,
    mut sprites: Query<&mut Transform, (With<Billboard>, Without<Camera>)>
) {
    if let Ok(camera_transform) = camera.get_single() {
        for mut sprite_transform in sprites.iter_mut() {
            sprite_transform.look_at(camera_transform.translation, Vec3::Y);
        }
    }
}
```

## Design Philosophy

You might wonder why Bevy stores transforms as translation + rotation + scale rather than as 4x4 matrices like some engines. The choice provides several practical advantages.

Repeated matrix multiplications accumulate floating-point errors over time. Storing the fundamental components avoids this numerical drift. The fields are directly accessible without accessor methods or matrix decomposition. The representation is more compact and faster to compute in hierarchies, since the matrix is only generated when needed for rendering.

Most importantly, the API feels natural. Programmers work with positions, rotations, and scales conceptually. Expressing these directly is more intuitive than manipulating matrices.

When the renderer needs a matrix representation, `GlobalTransform` computes it efficiently on demand.

## Performance Characteristics

Bevy optimizes transform operations behind the scenes. Change detection prevents recalculating transforms that haven't moved. Hierarchy propagation only updates subtrees that actually changed. The transform propagation system uses SIMD instructions for matrix operations where possible.

For static level geometry, transforms essentially have zero runtime cost after initial setup. For moving entities, the overhead is minimal compared to other frame costs like rendering or physics.

## Best Practices

A few guidelines will help you use transforms effectively:

Always modify `Transform`, never `GlobalTransform`. The propagation system will overwrite manual GlobalTransform changes.

When organizing static level geometry, parent it under a common root entity. This lets you move or rotate entire sections of the level as a unit.

Keep hierarchies reasonably shallow. While Bevy handles deep hierarchies correctly, very deep chains can impact performance. Aim for 3-4 levels of nesting at most unless there's a compelling reason for more.

For character movement, use the directional vector methods rather than manually calculating directions. They're clearer and automatically account for rotation.

Transforms form the foundation of spatial reasoning in Bevy. Master them, and you'll find positioning, orienting, and moving entities becomes second nature.

