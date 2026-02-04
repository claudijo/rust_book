# 3D Features

Bevy provides several features for 3D game development.

## GLTF Model Loading

**Bevy 0.1-0.2:**
Load GLTF files as Mesh assets (limited to first mesh with single texture):

```rust
commands.spawn(PbrComponents {
    mesh: asset_server.load("boat.gltf").unwrap(),
    material: materials.add(asset_server.load("boat.png").into()),
    ..Default::default()
});
```

**Bevy 0.3+ (GLTF Scene Loader):**

The GLTF loader was completely rewritten to load GLTF files as Bevy Scenes, including all meshes and textures:

```rust
fn load_gltf_system(mut commands: Commands, asset_server: Res<AssetServer>) {
    // Loads the entire GLTF file with all meshes and textures
    let scene_handle = asset_server.load("models/FlightHelmet/FlightHelmet.gltf");
    commands.spawn_scene(scene_handle);
}
```

This single line loads complex models like the Khronos Flight Helmet example, which consists of multiple meshes and textures. The new loader takes advantage of:
- Multi-asset loading capability
- Asset dependencies
- Sub-asset loading

You can also load specific parts:
```rust
// Load a specific mesh from the GLTF file
let mesh = asset_server.load("model.gltf#Mesh0/Primitive0");
```

## Depth Based Draw Order

Front-to-back drawing for fast "early fragment discarding" of opaque materials, and back-to-front drawing for correct transparent materials.

## Parenting

Parent transforms are propagated to their descendants:

```rust
commands
    .spawn(PbrComponents {
        mesh: cube_handle,
        ..Default::default()
    })
    .with_children(|parent| {
        parent.spawn(PbrComponents {
            mesh: cube_handle,
            translation: Translation::new(0.0, 2.0, 0.0),
            ..Default::default()
        });
    });
```

## MSAA

Get nice smooth edges by using Multi-Sample Anti-Aliasing:

```rust
app.add_resource(Msaa { samples: 8 })
```

## Bevy 0.2 Improvements

### Binary GLTF Support

**Added in Bevy 0.2**

Support for binary glTF (.glb) format was added.

### GLTF Loading Improvements

GLTFs can now be loaded that don't have UVs and normals, making the loader more flexible for various model types.

## Mesh Improvements in Bevy 0.3

**Added in Bevy 0.3**

### Flexible Vertex Attributes

Bevy meshes used to require exactly three vertex attributes: position, normal, and uv. Bevy 0.3 adds support for custom vertex attributes. Meshes can define whatever attributes they want and shaders can consume whatever attributes they want!

```rust
// Add custom vertex attributes
mesh.attributes.insert(
    "Vertex_Color",
    vec![[1.0, 0.0, 0.0], [0.0, 1.0, 0.0], [0.0, 0.0, 1.0]]
);
```

This enables:
- **Vertex colors** for per-vertex coloring
- **Bone weights** for animation
- **Custom data** for special effects
- Any other per-vertex data

For missing attributes (requested by shader but not defined by mesh), Bevy provides a zero-filled fallback buffer.

### Index Buffer Specialization

Rendering meshes often involves using vertex "indices" to reduce duplicate vertex data. Bevy used to hard-code these to u16, which was too small for some cases. 

**Bevy 0.3** allows render pipelines to specialize based on index buffer format:
- Now defaults to **u32** to cover most use cases
- Pipelines can handle both u16 and u32 indices
- Allows for larger, more complex meshes

