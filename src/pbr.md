# Physically Based Rendering (PBR)

**Added in Bevy 0.5**

Bevy now uses Physically Based Rendering (PBR) for 3D graphics. PBR is a semi-standard approach that attempts to use approximations of real-world "physically based" lighting and material properties.

## What is PBR?

PBR aims to render materials in a way that looks correct under any lighting condition by modeling:
- How light reflects off surfaces (specular reflection)
- How light scatters within materials (subsurface scattering approximations)
- How rough or smooth surfaces are (roughness)
- Whether materials are metallic or non-metallic (metallic property)

Bevy's implementation is based on:
- Filament PBR techniques (primary)
- Unreal Engine ideas
- Disney's principles

## StandardMaterial

The `StandardMaterial` is Bevy's PBR material with the following properties:

```rust
commands.spawn_bundle(PbrBundle {
    mesh: meshes.add(Mesh::from(shape::Cube { size: 1.0 })),
    material: materials.add(StandardMaterial {
        base_color: Color::rgb(0.8, 0.7, 0.6),
        perceptual_roughness: 0.5,  // Renamed from "roughness" in 0.6
        metallic: 0.0,
        reflectance: 0.5,
        emissive: Color::BLACK,
        ..Default::default()
    }),
    ..Default::default()
});
```

**Changed in Bevy 0.6:** The `roughness` field was renamed to `perceptual_roughness` to better reflect that it uses perceptual (not linear) roughness values.

### Material Properties

**base_color**: The base color of the material
- Represents the perceived color in neutral lighting
- Can be set as a solid color

**perceptual_roughness**: How rough the surface is (0.0 to 1.0)
- 0.0 = perfectly smooth/mirror-like
- 1.0 = completely rough/matte
- Uses perceptual roughness (more intuitive than linear)
- Affects how light scatters when reflecting

**metallic**: Whether the material is metallic (0.0 to 1.0)
- 0.0 = non-metal (dielectric)
- 1.0 = pure metal (conductor)
- Affects how light reflects and the appearance of reflections

**reflectance**: How reflective non-metallic surfaces are (0.0 to 1.0)
- Only affects non-metallic materials (metallic = 0.0)
- Common values: 0.5 for most materials
- Higher values = more reflective

**emissive**: Light emitted by the material
- Color and intensity of light the material emits
- Useful for glowing objects, screens, lights
- Does not actually illuminate other objects (just visual)

## Texture Support

StandardMaterial supports multiple texture types:

```rust
materials.add(StandardMaterial {
    base_color_texture: Some(asset_server.load("textures/color.png")),
    normal_map_texture: Some(asset_server.load("textures/normal.png")),
    metallic_roughness_texture: Some(asset_server.load("textures/metallic_roughness.png")),
    occlusion_texture: Some(asset_server.load("textures/occlusion.png")),
    emissive_texture: Some(asset_server.load("textures/emissive.png")),
    ..Default::default()
})
```

### Texture Types

**base_color_texture**: Base color/albedo map
- Contains the base color information
- RGB channels for color, optional A for transparency

**normal_map_texture**: Normal map for surface detail
- Adds surface detail without additional geometry
- Encodes surface normal directions in RGB

**metallic_roughness_texture**: Combined metallic and roughness
- Blue channel = metallic
- Green channel = roughness
- Efficient: combines two properties in one texture

**occlusion_texture**: Ambient occlusion
- Simulates soft shadows in crevices and corners
- Darkens areas where ambient light would be blocked

**emissive_texture**: Emissive/glow map
- Defines which parts of the material glow
- RGB for color and intensity

## Lighting

PBR materials require proper lighting to look their best.

### Point Lights

**Changed in Bevy 0.6:** `Light` and `LightBundle` were renamed to `PointLight` and `PointLightBundle`.

```rust
// Point light
commands.spawn_bundle(PointLightBundle {
    point_light: PointLight {
        intensity: 1500.0,
        color: Color::rgb(1.0, 0.9, 0.8),
        range: 20.0,
        ..Default::default()
    },
    transform: Transform::from_translation(Vec3::new(4.0, 5.0, 4.0)),
    ..Default::default()
});
```

### Directional Lights

```rust
// Directional light (sun)
commands.spawn_bundle(DirectionalLightBundle {
    directional_light: DirectionalLight {
        color: Color::rgb(1.0, 1.0, 1.0),
        illuminance: 10000.0,
        ..Default::default()
    },
    transform: Transform::from_rotation(Quat::from_euler(
        EulerRot::XYZ,
        -std::f32::consts::FRAC_PI_4,
        std::f32::consts::FRAC_PI_4,
        0.0,
    )),
    ..Default::default()
});
```

### Shadows

**Added in Bevy 0.6**

Both directional and point lights now support shadows!

```rust
// Point light with shadows
commands.spawn_bundle(PointLightBundle {
    point_light: PointLight {
        intensity: 1500.0,
        shadows_enabled: true,  // Enable shadows
        ..Default::default()
    },
    ..Default::default()
});

// Directional light with shadows
commands.spawn_bundle(DirectionalLightBundle {
    directional_light: DirectionalLight {
        illuminance: 10000.0,
        shadows_enabled: true,  // Enable shadows
        ..Default::default()
    },
    ..Default::default()
});
```

Shadows significantly enhance realism by showing where objects block light. The new renderer in 0.6 supports efficient shadow mapping for both light types.

## PBR Example

Bevy includes a PBR example that demonstrates different material properties:

```rust
fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    let sphere_mesh = meshes.add(Mesh::from(shape::Icosphere {
        radius: 1.0,
        subdivisions: 32,
    }));

    // Create spheres with varying metallic and roughness values
    for metallic in 0..4 {
        for roughness in 0..4 {
            commands.spawn_bundle(PbrBundle {
                mesh: sphere_mesh.clone(),
                material: materials.add(StandardMaterial {
                    base_color: Color::rgb(0.8, 0.7, 0.6),
                    metallic: metallic as f32 / 3.0,
                    roughness: roughness as f32 / 3.0,
                    ..Default::default()
                }),
                transform: Transform::from_xyz(
                    metallic as f32 * 2.5 - 3.75,
                    0.0,
                    roughness as f32 * 2.5 - 3.75,
                ),
                ..Default::default()
            });
        }
    }

    // Add a light
    commands.spawn_bundle(LightBundle {
        transform: Transform::from_translation(Vec3::new(0.0, 5.0, 5.0)),
        ..Default::default()
    });

    // Add a camera
    commands.spawn_bundle(PerspectiveCameraBundle {
        transform: Transform::from_xyz(0.0, 6.0, 12.0)
            .looking_at(Vec3::ZERO, Vec3::Y),
        ..Default::default()
    });
}
```

## Visual Quality

PBR provides:
- **Consistent appearance** across different lighting conditions
- **Physically accurate** light interactions
- **Energy conservation** (materials don't reflect more light than they receive)
- **Predictable results** when authoring materials
- **Industry standard** workflow (same as Unreal, Unity, etc.)

## Tips for Good PBR Materials

1. **Use reference images** - Real-world materials for inspiration
2. **Roughness is key** - Few real materials are perfectly smooth
3. **Metallic is binary** - Materials are usually 0.0 or 1.0, rarely in between
4. **Mind your albedo** - Base colors should be in realistic ranges
5. **Use proper texture maps** - Normal maps add detail without geometry cost
6. **Light your scenes well** - PBR needs good lighting to shine

PBR is the foundation of modern 3D rendering in Bevy and enables creating visually impressive, realistic 3D games and applications.

