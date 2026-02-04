# 3D Features

Bevy provides several features for 3D game development.

## GLTF Model Loading

Load GLTF files as Mesh assets:

```rust
commands.spawn(PbrComponents {
    // load the model
    mesh: asset_server.load("boat.gltf").unwrap(),
    // create a material for the model
    material: materials.add(asset_server.load("boat.png").into()),
    ..Default::default()
});
```

> **Note**: In the near future we will add support for loading GLTF files as Scenes instead of meshes.

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

