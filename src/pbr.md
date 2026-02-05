# Physically Based Rendering

Bevy uses Physically Based Rendering (PBR) for 3D graphics. PBR simulates how light behaves in the real world, producing materials that look correct under any lighting condition.

## Understanding PBR

Traditional rendering uses ad-hoc formulas that work for specific cases but break down in complex lighting. PBR takes a different approach: model the physics of light interaction with surfaces. This creates materials that respond realistically to different light sources and environments.

The technique models several key properties:
- How light reflects off surfaces based on viewing angle
- How surface roughness scatters reflected light
- The difference between metals and non-metals
- Energy conservation - surfaces never reflect more light than they receive

This physical accuracy means artists can create materials that look right in any scene without per-lighting tweaks.

## StandardMaterial

Bevy's PBR material provides properties that control appearance:

```rust
fn spawn_metal_sphere(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>
) {
    commands.spawn(PbrBundle {
        mesh: meshes.add(Mesh::from(shape::UVSphere {
            radius: 1.0,
            ..Default::default()
        })),
        material: materials.add(StandardMaterial {
            base_color: Color::rgb(0.8, 0.7, 0.6),
            metallic: 1.0,
            perceptual_roughness: 0.2,
            reflectance: 0.5,
            ..Default::default()
        }),
        transform: Transform::from_xyz(0.0, 1.0, 0.0),
        ..Default::default()
    });
}
```

## Material Properties

### Base Color

The fundamental color of the material in neutral lighting:

```rust
material.base_color = Color::rgb(0.8, 0.2, 0.2);  // Red material
```

This represents what color the surface appears to be. For metals, base color includes the metallic tint. For non-metals, it's the perceived color.

### Metallic

Controls whether the material behaves as metal or non-metal:

```rust
material.metallic = 0.0;  // Non-metal (wood, plastic, stone)
material.metallic = 1.0;  // Metal (iron, gold, copper)
```

Real-world materials are usually fully one or the other. Intermediate values exist for transitions (rust, weathering) but are rarely the base state.

Metals have these properties:
- Reflect colored light based on base color
- No diffuse reflection
- Strong specular highlights

Non-metals (dielectrics):
- Reflect white/colorless light
- Have diffuse reflection
- Weaker specular highlights

### Roughness

How smooth or rough the surface is:

```rust
material.perceptual_roughness = 0.0;  // Mirror smooth
material.perceptual_roughness = 0.5;  // Semi-rough
material.perceptual_roughness = 1.0;  // Completely matte
```

Smooth surfaces produce sharp, focused reflections. Rough surfaces scatter reflected light, creating broader, dimmer highlights. Few real materials are perfectly smooth - even polished metal has microscopic roughness.

### Reflectance

Controls how reflective non-metallic surfaces are:

```rust
material.reflectance = 0.5;  // Standard (most materials)
material.reflectance = 0.35; // Less reflective (cloth, leather)
material.reflectance = 0.7;  // More reflective (glass, water)
```

This only affects non-metals. Most common materials fall around 0.4-0.6.

### Emissive

Light emitted by the surface:

```rust
material.emissive = Color::rgb(1.0, 0.5, 0.0);  // Orange glow
```

Emissive materials appear to glow. This is visual only - they don't actually illuminate other objects. Use it for screens, light strips, glowing eyes, or magical effects.

## Textures

Apply textures to add detail:

```rust
let material = materials.add(StandardMaterial {
    base_color_texture: Some(asset_server.load("wood_color.png")),
    normal_map_texture: Some(asset_server.load("wood_normal.png")),
    metallic_roughness_texture: Some(asset_server.load("wood_mr.png")),
    occlusion_texture: Some(asset_server.load("wood_ao.png")),
    ..Default::default()
});
```

Each texture type serves a specific purpose:

**Base color texture** - The surface's color pattern (wood grain, brick pattern, etc.)

**Normal map** - Adds surface detail like bumps and grooves without changing geometry. The GPU uses this to modify lighting calculations per-pixel.

**Metallic/roughness texture** - Combines metallic (blue channel) and roughness (green channel) in one texture. This lets different areas have different material properties.

**Occlusion texture** - Simulates ambient shadows in crevices and corners. Darkens areas where ambient light would be blocked.

**Emissive texture** - Defines which parts glow and their color.

## Lighting for PBR

PBR materials need proper lighting to show their properties. Use multiple light types:

```rust
fn setup_lighting(mut commands: Commands) {
    // Directional light (sun)
    commands.spawn(DirectionalLightBundle {
        directional_light: DirectionalLight {
            illuminance: 10000.0,
            shadows_enabled: true,
            ..Default::default()
        },
        transform: Transform::from_rotation(Quat::from_euler(
            EulerRot::XYZ,
            -0.7,
            0.5,
            0.0,
        )),
        ..Default::default()
    });
    
    // Fill light for softer shadows
    commands.spawn(PointLightBundle {
        point_light: PointLight {
            intensity: 2000.0,
            color: Color::rgb(1.0, 0.9, 0.8),
            ..Default::default()
        },
        transform: Transform::from_xyz(-5.0, 5.0, 5.0),
        ..Default::default()
    });
}
```

Shadows significantly enhance realism by showing spatial relationships between objects.

## Visualizing Material Properties

Create a grid of spheres showing different metallic and roughness combinations:

```rust
fn spawn_material_grid(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>
) {
    let sphere_mesh = meshes.add(Mesh::from(shape::UVSphere {
        radius: 0.4,
        sectors: 32,
        stacks: 16,
    }));
    
    for x in 0..5 {
        for z in 0..5 {
            let metallic = x as f32 / 4.0;
            let roughness = z as f32 / 4.0;
            
            commands.spawn(PbrBundle {
                mesh: sphere_mesh.clone(),
                material: materials.add(StandardMaterial {
                    base_color: Color::rgb(0.8, 0.7, 0.6),
                    metallic,
                    perceptual_roughness: roughness,
                    ..Default::default()
                }),
                transform: Transform::from_xyz(
                    x as f32 * 2.0 - 4.0,
                    0.0,
                    z as f32 * 2.0 - 4.0,
                ),
                ..Default::default()
            });
        }
    }
}
```

This creates a 5x5 grid. Each row has constant metallic values from 0.0 to 1.0. Each column has constant roughness from 0.0 to 1.0. The resulting grid shows how these properties interact.

## Common Materials

**Polished metal:**
```rust
StandardMaterial {
    base_color: Color::rgb(0.7, 0.7, 0.7),
    metallic: 1.0,
    perceptual_roughness: 0.1,
    ..Default::default()
}
```

**Rough stone:**
```rust
StandardMaterial {
    base_color: Color::rgb(0.5, 0.5, 0.5),
    metallic: 0.0,
    perceptual_roughness: 0.9,
    ..Default::default()
}
```

**Painted plastic:**
```rust
StandardMaterial {
    base_color: Color::rgb(0.8, 0.2, 0.2),
    metallic: 0.0,
    perceptual_roughness: 0.3,
    ..Default::default()
}
```

**Glowing screen:**
```rust
StandardMaterial {
    base_color: Color::BLACK,
    emissive: Color::rgb(0.0, 0.8, 1.0),
    ..Default::default()
}
```

## Best Practices

**Use reference photos** - Study real materials to understand their properties. Metals have colored reflections. Most materials are rougher than you think.

**Roughness variation** - Real surfaces have varying roughness. Use roughness maps to add detail even to simple geometry.

**Conservative albedo** - Base colors should fall in realistic ranges. Pure white or black materials are rare in nature.

**Metallic is binary** - Most materials are clearly metal or non-metal. Only use intermediate values for transitions like rust or corrosion.

**Light your scenes** - PBR needs varied lighting to show material properties. Use multiple light sources with different colors and intensities.

**Test under different lighting** - Materials should look good in various lighting conditions. That's the promise of PBR.

## Technical Implementation

Bevy's PBR implementation draws from industry-standard techniques:
- Cook-Torrance specular BRDF (Bidirectional Reflectance Distribution Function)
- Disney diffuse BRDF
- Image-based lighting support
- Screen-space reflections (when available)

These techniques ensure materials behave correctly under complex lighting while remaining computationally efficient.

PBR is the foundation of realistic 3D rendering in Bevy. Understanding metallic, roughness, and proper lighting lets you create convincing materials for any type of 3D game or application.

