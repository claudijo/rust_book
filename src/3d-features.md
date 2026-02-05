# 3D Rendering

Bevy provides complete 3D rendering capabilities: mesh loading, materials, lighting, and cameras. Understanding these building blocks lets you create rich 3D worlds.

## 3D Entities

A 3D entity needs a mesh (the shape), a material (how it looks), and a transform (where it is). Create one with a PbrBundle:

```rust
fn spawn_cube(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>
) {
    commands.spawn(PbrBundle {
        mesh: meshes.add(Mesh::from(shape::Cube { size: 1.0 })),
        material: materials.add(Color::rgb(0.8, 0.7, 0.6).into()),
        transform: Transform::from_xyz(0.0, 0.5, 0.0),
        ..Default::default()
    });
}
```

This spawns a cube at position (0, 0.5, 0) with a tan color.

## Meshes

Meshes define 3D shapes through vertices and triangles. Bevy includes common primitives:

```rust
// Basic shapes
let cube = Mesh::from(shape::Cube { size: 1.0 });
let sphere = Mesh::from(shape::UVSphere { radius: 0.5, ..Default::default() });
let plane = Mesh::from(shape::Plane { size: 10.0 });
let cylinder = Mesh::from(shape::Cylinder { radius: 0.5, height: 2.0, ..Default::default() });
```

Add these to the mesh assets collection to use them:

```rust
let cube_handle = meshes.add(cube);
```

## Custom Meshes

Create custom meshes by defining vertices and indices:

```rust
fn create_triangle(mut meshes: ResMut<Assets<Mesh>>) {
    let mut mesh = Mesh::new(PrimitiveTopology::TriangleList);
    
    // Positions
    mesh.insert_attribute(
        Mesh::ATTRIBUTE_POSITION,
        vec![
            [0.0, 1.0, 0.0],   // Top
            [-1.0, -1.0, 0.0], // Bottom left
            [1.0, -1.0, 0.0],  // Bottom right
        ],
    );
    
    // Normals (pointing toward camera)
    mesh.insert_attribute(
        Mesh::ATTRIBUTE_NORMAL,
        vec![[0.0, 0.0, 1.0]; 3],
    );
    
    // UV coordinates for texturing
    mesh.insert_attribute(
        Mesh::ATTRIBUTE_UV_0,
        vec![
            [0.5, 1.0],
            [0.0, 0.0],
            [1.0, 0.0],
        ],
    );
    
    // Indices define triangle winding order
    mesh.set_indices(Some(Indices::U32(vec![0, 1, 2])));
    
    let handle = meshes.add(mesh);
}
```

Meshes support custom vertex attributes beyond position, normal, and UV. Add vertex colors, bone weights, or any per-vertex data your shader needs.

## GLTF Models

GLTF is the standard format for 3D models. Load entire scenes with all their meshes, textures, and materials:

```rust
fn load_model(mut commands: Commands, asset_server: Res<AssetServer>) {
    let scene = asset_server.load("models/spaceship.gltf#Scene0");
    commands.spawn(SceneBundle {
        scene,
        ..Default::default()
    });
}
```

GLTF files can contain multiple assets. Load specific parts using labels:

```rust
// Load just a specific mesh
let mesh = asset_server.load("model.gltf#Mesh0/Primitive0");

// Load a specific material
let material = asset_server.load("model.gltf#Material0");

// Load the entire scene
let scene = asset_server.load("model.gltf#Scene0");
```

The asset system handles dependencies automatically - loading a scene loads all referenced meshes and textures.

## Materials

Materials define how surfaces respond to light. The StandardMaterial provides physically-based rendering:

```rust
let material = materials.add(StandardMaterial {
    base_color: Color::rgb(0.8, 0.7, 0.6),
    metallic: 0.0,        // 0.0 = non-metal, 1.0 = metal
    perceptual_roughness: 0.5,  // 0.0 = smooth, 1.0 = rough
    reflectance: 0.5,     // How much light reflects
    ..Default::default()
});
```

### Textures

Apply textures to materials:

```rust
let material = materials.add(StandardMaterial {
    base_color_texture: Some(asset_server.load("textures/wood.png")),
    normal_map_texture: Some(asset_server.load("textures/wood_normal.png")),
    metallic_roughness_texture: Some(asset_server.load("textures/wood_mr.png")),
    ..Default::default()
});
```

Different texture types serve different purposes:
- **Base color** - The surface's color
- **Normal map** - Adds surface detail without geometry
- **Metallic/roughness** - Controls material properties
- **Emissive** - Makes surfaces glow
- **Occlusion** - Adds ambient shadows

## Lighting

Lights make 3D scenes visible. Bevy supports several light types:

### Point Lights

Emit light in all directions from a point:

```rust
commands.spawn(PointLightBundle {
    point_light: PointLight {
        intensity: 1500.0,
        radius: 10.0,
        color: Color::rgb(1.0, 0.9, 0.8),
        ..Default::default()
    },
    transform: Transform::from_xyz(4.0, 8.0, 4.0),
    ..Default::default()
});
```

### Directional Lights

Simulate sunlight - parallel rays from a direction:

```rust
commands.spawn(DirectionalLightBundle {
    directional_light: DirectionalLight {
        illuminance: 10000.0,
        color: Color::rgb(1.0, 0.95, 0.9),
        ..Default::default()
    },
    transform: Transform::from_rotation(Quat::from_euler(
        EulerRot::XYZ,
        -std::f32::consts::PI / 4.0,
        std::f32::consts::PI / 4.0,
        0.0,
    )),
    ..Default::default()
});
```

### Spot Lights

Emit light in a cone:

```rust
commands.spawn(SpotLightBundle {
    spot_light: SpotLight {
        intensity: 2000.0,
        range: 20.0,
        inner_angle: 0.3,
        outer_angle: 0.8,
        color: Color::WHITE,
        ..Default::default()
    },
    transform: Transform::from_xyz(0.0, 5.0, 0.0)
        .looking_at(Vec3::ZERO, Vec3::Y),
    ..Default::default()
});
```

## Cameras

3D scenes need a camera to define the viewpoint:

```rust
commands.spawn(Camera3dBundle {
    transform: Transform::from_xyz(0.0, 5.0, 10.0)
        .looking_at(Vec3::ZERO, Vec3::Y),
    ..Default::default()
});
```

The camera's transform determines what the player sees. Move and rotate it to control the view.

### Perspective vs Orthographic

Perspective cameras (the default) have realistic depth:

```rust
commands.spawn(Camera3dBundle {
    projection: PerspectiveProjection {
        fov: std::f32::consts::PI / 4.0,
        near: 0.1,
        far: 1000.0,
        ..Default::default()
    }
    .into(),
    ..Default::default()
});
```

Orthographic cameras have no perspective distortion:

```rust
commands.spawn(Camera3dBundle {
    projection: OrthographicProjection {
        scale: 3.0,
        ..Default::default()
    }
    .into(),
    ..Default::default()
});
```

Orthographic is useful for strategy games, editors, or artistic styles.

## Anti-Aliasing

Smooth jagged edges with MSAA (Multi-Sample Anti-Aliasing):

```rust
fn main() {
    App::new()
        .insert_resource(Msaa::Sample4)  // 4x MSAA
        .add_plugins(DefaultPlugins)
        .run();
}
```

Higher sample counts (Sample8) improve quality but cost performance. Sample4 is usually a good balance.

## Depth and Transparency

Opaque objects render front-to-back for efficiency. The depth buffer discards hidden fragments early, saving GPU work.

Transparent objects render back-to-front for correct blending. Enable transparency in materials:

```rust
let material = materials.add(StandardMaterial {
    base_color: Color::rgba(1.0, 0.0, 0.0, 0.5),  // 50% transparent red
    alpha_mode: AlphaMode::Blend,
    ..Default::default()
});
```

Transparent rendering is more expensive than opaque - use it judiciously.

## Hierarchies

Parent-child relationships work in 3D just like 2D. Attach objects to others:

```rust
// Parent entity
let parent = commands.spawn(PbrBundle {
    mesh: cube_mesh.clone(),
    material: parent_material.clone(),
    ..Default::default()
})
.id();

// Child entity - positioned relative to parent
commands.entity(parent).with_children(|parent| {
    parent.spawn(PbrBundle {
        mesh: sphere_mesh.clone(),
        material: child_material.clone(),
        transform: Transform::from_xyz(2.0, 0.0, 0.0),
        ..Default::default()
    });
});
```

If the parent rotates, children orbit around it while maintaining relative positions.

## Common Patterns

**Third-person camera** - Follow a character:

```rust
fn camera_follow(
    player: Query<&Transform, With<Player>>,
    mut camera: Query<&mut Transform, (With<Camera3d>, Without<Player>)>
) {
    if let Ok(player_transform) = player.get_single() {
        for mut camera_transform in camera.iter_mut() {
            let target = player_transform.translation + Vec3::new(0.0, 5.0, -10.0);
            camera_transform.translation = camera_transform.translation.lerp(target, 0.1);
            camera_transform.look_at(player_transform.translation, Vec3::Y);
        }
    }
}
```

**Billboards** - Make sprites always face the camera:

```rust
fn billboard(
    camera: Query<&Transform, With<Camera3d>>,
    mut billboards: Query<&mut Transform, (With<Billboard>, Without<Camera3d>)>
) {
    if let Ok(camera_transform) = camera.get_single() {
        for mut transform in billboards.iter_mut() {
            transform.look_at(camera_transform.translation, Vec3::Y);
        }
    }
}
```

**Simple rotation** - Spin objects:

```rust
fn rotate_objects(time: Res<Time>, mut query: Query<&mut Transform, With<Spinning>>) {
    for mut transform in query.iter_mut() {
        transform.rotate_y(time.delta_seconds());
    }
}
```

## Performance Tips

**Use LOD (Level of Detail)** - Simpler meshes for distant objects.

**Frustum culling** - Bevy automatically skips rendering objects outside the camera's view.

**Batch by material** - Objects sharing materials render together efficiently.

**Minimize transparent objects** - They're more expensive than opaque rendering.

**Profile first** - Measure before optimizing. Many games run fine without complex optimizations.

Bevy's 3D rendering builds on standard graphics techniques. Understanding meshes, materials, and lighting will let you create any 3D experience you envision.

