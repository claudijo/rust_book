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
// Basic 3D shapes
let cube = Mesh::from(shape::Cube { size: 1.0 });
let sphere = Mesh::from(shape::UVSphere { radius: 0.5, ..Default::default() });
let plane = Mesh::from(shape::Plane { size: 10.0 });
let cylinder = Mesh::from(shape::Cylinder { radius: 0.5, height: 2.0, ..Default::default() });

// 2D shapes (useful for sprites and UI)
let circle = Mesh::from(shape::Circle::new(10.0));
let pentagon = Mesh::from(shape::RegularPolygon::new(10.0, 5));
let hexagon = Mesh::from(shape::RegularPolygon::new(10.0, 6));
```

Regular polygons create n-sided shapes with a specified radius and number of sides. Use these with `ColorMesh2dBundle` for 2D shapes or with standard materials for 3D.

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

### Flexible Vertex Layouts

Meshes support custom vertex attributes beyond position, normal, and UV. Bevy's shader system automatically specializes based on what attributes your mesh provides.

This flexibility means:

**Materials work with any mesh** - StandardMaterial and custom materials automatically adapt to whatever vertex attributes are available. Missing attributes degrade gracefully.

**Custom attributes supported** - Add vertex colors, bone weights, tangents, or any per-vertex data:

```rust
// Add vertex colors - fully supported by StandardMaterial and ColorMaterial
mesh.insert_attribute(
    Mesh::ATTRIBUTE_COLOR,
    vec![[1.0, 0.0, 0.0, 1.0]; vertex_count]  // RGBA
);

// Add custom attribute for special effects
mesh.insert_attribute(
    MeshVertexAttribute::new("VertexMetallic", 2988540917, VertexFormat::Float32),
    vec![0.5f32; vertex_count]
);
```

Vertex colors are particularly useful for procedural meshes, terrain, particle effects, or any scenario where you want per-vertex variation without textures. Both 2D and 3D materials automatically use vertex colors when present.

**Automatic tangent generation** - If a mesh has normal maps but lacks tangent data, Bevy automatically generates tangents using the industry-standard MikkTSpace algorithm. This ensures normal maps work correctly even on meshes that don't provide tangents.

**Shaders specialize automatically** - Bevy compiles shader variants based on mesh vertex layouts. A shader using vertex colors only compiles that path when rendering meshes with colors.

This system enables skeletal animation (bone weights and indices), custom rendering effects, and efficient mesh handling without hardcoded attribute requirements.

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

### Unlimited Point Lights

Bevy uses clustered forward rendering to efficiently handle many lights. On platforms with storage buffer support (everything except WebGL), scenes can have thousands of point lights:

```rust
fn spawn_many_lights(mut commands: Commands) {
    for x in -50..50 {
        for z in -50..50 {
            commands.spawn(PointLightBundle {
                point_light: PointLight {
                    intensity: 500.0,
                    radius: 5.0,
                    color: Color::rgb(
                        rand::random(),
                        rand::random(),
                        rand::random(),
                    ),
                    ..Default::default()
                },
                transform: Transform::from_xyz(
                    x as f32 * 2.0,
                    1.0,
                    z as f32 * 2.0,
                ),
                ..Default::default()
            });
        }
    }
}
```

Bevy automatically clusters lights spatially, calculating which lights affect each screen region. This enables scenes with 10,000+ lights while maintaining high performance.

### Clustered Rendering Configuration

Fine-tune light clustering for your scene:

```rust
use bevy::pbr::ClusterConfig;

commands.insert_resource(ClusterConfig::FixedZ {
    total: 4096,     // Total light clusters
    z_slices: 24,    // Depth slices
    z_config: Default::default(),
});
```

Configuration options:

**FixedZ** - Dynamic X/Y clusters, fixed Z slices. Adapts to light distribution automatically. Best for most scenes.

**Fixed** - Fixed cluster counts for all dimensions. Use when you know optimal cluster layout.

**Single** - One cluster for all lights. Use with very few lights.

**None** - Disable clustering. Use when you control light assignment manually.

The clustering system detects changes and only recalculates when lights move, optimizing CPU usage.

### Light Visibility

Toggle lights on and off with the Visibility component:

```rust
fn toggle_lights(
    keyboard: Res<Input<KeyCode>>,
    mut lights: Query<&mut Visibility, With<PointLight>>
) {
    if keyboard.just_pressed(KeyCode::L) {
        for mut visibility in lights.iter_mut() {
            visibility.is_visible = !visibility.is_visible;
        }
    }
}
```

### Visibility Inheritance

Visibility now propagates down entity hierarchies, similar to Transform. When you hide a parent entity, all its descendants automatically hide as well. This is invaluable for complex entities with many nested parts.

Consider a player entity with nested components: character mesh, equipment, weapons, and visual effects. Hiding the top-level player entity now automatically hides everything beneath it - no need to manually hide each piece.

To set visibility:

```rust
// Hide an entity and all its children
entity_visibility.is_visible = false;
```

To check if something is actually visible (considering parent visibility), use `ComputedVisibility`:

```rust
fn check_visibility(query: Query<&ComputedVisibility>) {
    for visibility in query.iter() {
        if visibility.is_visible() {
            // This entity is visible, considering parent visibility
        }
    }
}
```

Use `Visibility` to control an entity's visibility. Use `ComputedVisibility` to check the final visibility state after hierarchy propagation.

Invisible lights don't contribute to rendering, saving GPU time. Use this for day/night cycles, power-off states, or performance optimization.

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

### Camera-Driven Rendering

Each camera controls what it renders, how it renders, and where it renders to. Cameras run their own render graph to a specified render target (window or texture). This makes advanced rendering scenarios simple.

### Render To Texture

Rendering a camera to a texture is now a single field assignment:

```rust
fn setup_portal_camera(
    mut commands: Commands,
    mut images: ResMut<Assets<Image>>,
) {
    // Create target texture
    let size = Extent3d {
        width: 512,
        height: 512,
        ..Default::default()
    };
    
    let mut image = Image {
        texture_descriptor: TextureDescriptor {
            size,
            mip_level_count: 1,
            sample_count: 1,
            dimension: TextureDimension::D2,
            format: TextureFormat::Bgra8UnormSrgb,
            usage: TextureUsages::TEXTURE_BINDING
                | TextureUsages::COPY_DST
                | TextureUsages::RENDER_ATTACHMENT,
        },
        ..Default::default()
    };
    
    let image_handle = images.add(image);
    
    // Camera renders to texture
    commands.spawn(Camera3dBundle {
        camera: Camera {
            target: RenderTarget::Image(image_handle.clone()),
            ..Default::default()
        },
        ..Default::default()
    });
}
```

Use this for portals, security cameras, minimaps, UI textures in 3D space, player portraits, or any scenario where you need to render from another perspective.

### Split Screen

Create split-screen rendering by setting viewports on multiple cameras:

```rust
fn setup_split_screen(mut commands: Commands) {
    // Left camera - renders to left half
    commands.spawn(Camera3dBundle {
        camera: Camera {
            viewport: Some(Viewport {
                physical_position: UVec2::new(0, 0),
                physical_size: UVec2::new(960, 1080),
                ..Default::default()
            }),
            ..Default::default()
        },
        transform: Transform::from_xyz(-5.0, 5.0, 5.0)
            .looking_at(Vec3::ZERO, Vec3::Y),
        ..Default::default()
    });
    
    // Right camera - renders to right half
    commands.spawn(Camera3dBundle {
        camera: Camera {
            viewport: Some(Viewport {
                physical_position: UVec2::new(960, 0),
                physical_size: UVec2::new(960, 1080),
                ..Default::default()
            }),
            ..Default::default()
        },
        transform: Transform::from_xyz(5.0, 5.0, 5.0)
            .looking_at(Vec3::ZERO, Vec3::Y),
        ..Default::default()
    });
}
```

The viewport defines which section of the render target each camera draws to. Perfect for local multiplayer or picture-in-picture effects.

### Camera Layering and Priority

Layer cameras on top of each other using the priority field:

```rust
fn setup_layered_cameras(mut commands: Commands) {
    // Base camera - renders first
    commands.spawn(Camera3dBundle {
        camera: Camera {
            priority: 0,  // Lower priority renders first
            ..Default::default()
        },
        ..Default::default()
    });
    
    // Overlay camera - renders on top
    commands.spawn(Camera3dBundle {
        camera_3d: Camera3d {
            // Don't clear - preserve base camera's render
            clear_color: ClearColorConfig::None,
            ..Default::default()
        },
        camera: Camera {
            priority: 1,  // Higher priority renders last
            ..Default::default()
        },
        ..Default::default()
    });
}
```

Higher priority cameras render after lower priority cameras. Use `ClearColorConfig::None` to preserve previous renders and layer on top. This enables custom UI passes, minimaps overlaid on the main view, or multi-pass rendering effects.

### Enabling and Disabling Cameras

Control whether a camera renders using the `is_active` field:

```rust
fn toggle_camera(
    keyboard: Res<Input<KeyCode>>,
    mut cameras: Query<&mut Camera>
) {
    if keyboard.just_pressed(KeyCode::C) {
        for mut camera in cameras.iter_mut() {
            camera.is_active = !camera.is_active;
        }
    }
}
```

Inactive cameras don't render, saving GPU time. Use this for toggling between different views or temporarily disabling rendering.

### Camera Target Size

Cameras store their render target size locally for easy access:

```rust
fn use_camera_size(cameras: Query<&Camera>) {
    for camera in cameras.iter() {
        if let Some(size) = camera.logical_target_size() {
            println!("Camera renders to {}x{}", size.x, size.y);
        }
        
        if let Some(viewport_size) = camera.logical_viewport_size() {
            println!("Viewport size: {}x{}", viewport_size.x, viewport_size.y);
        }
    }
}
```

World-to-screen conversion is also simpler:

```rust
fn world_to_screen(
    cameras: Query<(&Camera, &GlobalTransform)>
) {
    for (camera, transform) in cameras.iter() {
        let world_pos = Vec3::new(0.0, 1.0, 0.0);
        
        if let Some(screen_pos) = camera.world_to_viewport(transform, world_pos) {
            println!("World position projects to screen: {:?}", screen_pos);
        }
    }
}
```

### RenderLayers

Control which entities each camera renders using RenderLayers:

```rust
fn setup_layered_rendering(mut commands: Commands) {
    // Camera only sees layer 0
    commands.spawn(Camera3dBundle {
        camera: Camera {
            target: RenderTarget::Image(texture_handle),
            ..Default::default()
        },
        ..Default::default()
    })
    .insert(RenderLayers::layer(0));
    
    // Main camera sees layer 1
    commands.spawn(Camera3dBundle::default())
        .insert(RenderLayers::layer(1));
    
    // Entity on layer 0 - only rendered by first camera
    commands.spawn(PbrBundle {
        mesh: mesh_handle,
        material: material_handle,
        ..Default::default()
    })
    .insert(RenderLayers::layer(0));
    
    // Entity on layer 1 - only rendered by main camera
    commands.spawn(PbrBundle {
        mesh: mesh_handle,
        material: material_handle,
        ..Default::default()
    })
    .insert(RenderLayers::layer(1));
}
```

Entities without RenderLayers render on all cameras. Entities with RenderLayers only render on cameras with matching layers. Use this to render different sets of objects to different cameras, perfect for portals, minimaps, or hiding UI from screenshots.

### Custom Render Graphs (Advanced)

For specialized rendering needs, cameras can use custom render graphs instead of the default 2D or 3D graphs:

```rust
commands.spawn(Camera3dBundle {
    camera_render_graph: CameraRenderGraph::new(some_custom_graph),
    ..Default::default()
});
```

This enables completely custom rendering pipelines, like replacing forward rendering with deferred rendering. Most scenarios don't need this - high-level Materials and extending the built-in render graphs cover common cases. Using the default render graph ensures compatibility with other plugins.

## Compute Shaders

Compute shaders perform general-purpose GPU computation outside the traditional rendering pipeline. Use them for physics simulations, particle systems, procedural generation, or any parallel computation that benefits from GPU acceleration.

Bevy's compute shader support includes hot reloading, shader imports, and shader defs just like regular shaders:

```wgsl
#import "shaders/common.wgsl"

@group(0) @binding(0) var<storage, read> input_data: array<f32>;
@group(0) @binding(1) var<storage, read_write> output_data: array<f32>;

@compute @workgroup_size(8, 8, 1)
fn main(@builtin(global_invocation_id) invocation_id: vec3<u32>) {
    let index = invocation_id.x + invocation_id.y * 256u;
    
    // Shader defs can be configured at runtime
    #ifdef DOUBLE_VALUE
        output_data[index] = input_data[index] * 2.0;
    #else
        output_data[index] = input_data[index];
    #endif
}
```

### Setting Up Compute Pipelines

Create a compute pipeline similar to render pipelines:

```rust
fn setup_compute(
    mut commands: Commands,
    mut pipelines: ResMut<Assets<ComputePipeline>>,
    shader_server: Res<AssetServer>
) {
    let shader = shader_server.load("shaders/compute.wgsl");
    
    let pipeline = pipelines.add(ComputePipeline {
        shader,
        shader_defs: vec!["DOUBLE_VALUE".to_string()],
        entry_point: "main".into(),
    });
}
```

Shader defs enable runtime pipeline specialization. Change defs to compile different shader variants dynamically.

### Use Cases

**Physics simulation** - Calculate forces and update positions on the GPU.

**Particle systems** - Update thousands of particles in parallel.

**Procedural generation** - Generate terrain, textures, or meshes on the GPU.

**Image processing** - Apply filters, effects, or analysis to textures.

**Pathfinding** - Compute navigation fields or flow maps.

Compute shaders excel at data-parallel operations where thousands of work items execute the same code on different data. The GPU's massively parallel architecture can outperform CPUs by orders of magnitude for suitable workloads.

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


**Simple rotation** - Spin objects continuously:

```rust
// First, define a marker component for objects that should spin
#[derive(Component)]
struct Spinning;

fn rotate_objects(time: Res<Time>, mut query: Query<&mut Transform, With<Spinning>>) {
    for mut transform in query.iter_mut() {
        transform.rotate_y(time.delta_seconds());
    }
}
```

## Performance Tips

**Use LOD (Level of Detail)** - Simpler meshes for distant objects.

**Frustum culling** - Bevy automatically skips rendering objects outside the camera's view. Frustum culling now runs in parallel across multiple CPU cores, significantly improving performance in scenes with many entities.

**Batch by material** - Objects sharing materials render together efficiently.

**Minimize transparent objects** - They're more expensive than opaque rendering.

**Profile first** - Measure before optimizing. Many games run fine without complex optimizations.

Bevy's 3D rendering builds on standard graphics techniques. Understanding meshes, materials, and lighting will let you create any 3D experience you envision.

