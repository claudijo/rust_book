# Materials and Custom Shaders

Materials define how surfaces look by controlling their interaction with light and how they render. Bevy provides a powerful material system that makes custom shaders easy to create while maintaining the performance and flexibility needed for complex rendering effects.

## The Material System

Bevy's material system combines Rust types with GPU shaders. Materials are Rust structs that derive `AsBindGroup` to automatically generate the GPU binding code, and implement the `Material` trait to define rendering behavior.

### Creating a Custom Material

Define a custom material with minimal boilerplate:

```rust
use bevy::prelude::*;
use bevy::reflect::TypeUuid;
use bevy::render::render_resource::{AsBindGroup, ShaderRef};

#[derive(AsBindGroup, TypeUuid, Clone)]
#[uuid = "f690fdae-d598-45ab-8225-97e2a3f056e0"]
pub struct CoolMaterial {
    #[uniform(0)]
    color: Color,
    #[texture(1)]
    #[sampler(2)]
    color_texture: Handle<Image>,
}

impl Material for CoolMaterial {
    fn fragment_shader() -> ShaderRef {
        "shaders/cool_material.wgsl".into()
    }
}
```

The corresponding WGSL shader (`shaders/cool_material.wgsl`):

```wgsl
struct CoolMaterial {
    color: vec4<f32>,
};

@group(1) @binding(0)
var<uniform> material: CoolMaterial;
@group(1) @binding(1)
var color_texture: texture_2d<f32>;
@group(1) @binding(2)
var color_sampler: sampler;

@fragment
fn fragment(
    #import bevy_pbr::mesh_vertex_output
) -> @location(0) vec4<f32> {
    return material.color * textureSample(color_texture, color_sampler, uv);
}
```

That's it! The `AsBindGroup` derive handles all GPU data conversion, writing to GPU buffers, and creating bind groups automatically.

### Using Custom Materials

Add the material plugin and spawn entities with your material:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugin(MaterialPlugin::<CoolMaterial>::default())
        .add_startup_system(setup)
        .run();
}

fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<CoolMaterial>>,
    asset_server: Res<AssetServer>,
) {
    let material = materials.add(CoolMaterial {
        color: Color::rgb(0.8, 0.2, 0.3),
        color_texture: asset_server.load("textures/pattern.png"),
    });
    
    commands.spawn(MaterialMeshBundle {
        mesh: meshes.add(Mesh::from(shape::Cube { size: 1.0 })),
        material,
        transform: Transform::from_xyz(0.0, 0.5, 0.0),
        ..Default::default()
    });
}
```

## AsBindGroup Attributes

The `AsBindGroup` derive supports various attributes for controlling GPU bindings:

### Uniforms

Uniforms provide read-only data to shaders:

```rust
#[derive(AsBindGroup)]
struct MyMaterial {
    #[uniform(0)]
    time: f32,
    #[uniform(0)]  // Same binding - combined into one buffer
    speed: f32,
}
```

Multiple fields with the same binding number are packed into a single uniform buffer, improving efficiency.

### Textures and Samplers

Textures require both a texture and sampler binding:

```rust
#[derive(AsBindGroup)]
struct MyMaterial {
    #[texture(0)]
    #[sampler(1)]
    albedo: Handle<Image>,
    
    #[texture(2, dimension = "2d_array")]
    #[sampler(3)]
    array_texture: Handle<Image>,
}
```

Configure texture dimensions, sample types, and filtering with attributes.

### Storage Buffers

For larger or dynamic data, use storage buffers:

```rust
#[derive(AsBindGroup)]
struct MyMaterial {
    #[storage(0, read_only)]
    data: Vec<Vec4>,
}
```

Storage buffers support much larger data than uniforms and can be read-write.

## Material Trait Configuration

The `Material` trait provides hooks for customizing material behavior:

```rust
impl Material for MyMaterial {
    fn fragment_shader() -> ShaderRef {
        "shaders/my_material.wgsl".into()
    }
    
    fn vertex_shader() -> ShaderRef {
        // Default uses built-in vertex shader
        ShaderRef::Default
    }
    
    fn alpha_mode(&self) -> AlphaMode {
        AlphaMode::Blend
    }
    
    fn depth_bias(&self) -> f32 {
        0.0
    }
    
    // Many more configuration options available
}
```

Override only what you need - everything has sensible defaults.

## Shader Imports and Modularization

Bevy's built-in shaders are modularized and importable. Reuse PBR lighting, mesh data, and view bindings in custom shaders:

```wgsl
#import bevy_pbr::utils
#import bevy_pbr::clustered_forward
#import bevy_pbr::lighting
#import bevy_pbr::shadows
#import bevy_pbr::pbr_types
#import bevy_pbr::pbr_functions

@fragment
fn fragment(in: FragmentInput) -> @location(0) vec4<f32> {
    var pbr_input: PbrInput = pbr_input_new();
    
    // Customize PBR input
    pbr_input.material.base_color = vec4<f32>(1.0, 0.0, 0.0, 1.0);
    pbr_input.material.metallic = 0.8;
    pbr_input.material.perceptual_roughness = 0.2;
    
    // Use built-in PBR lighting calculation
    return tone_mapping(pbr(pbr_input));
}
```

This enables creating custom PBR materials without reimplementing lighting calculations. Extend the built-in PBR logic rather than replace it.

### Common Imports

**`bevy_pbr::mesh_vertex_output`** - Mesh vertex data (position, normal, UV, etc.)

**`bevy_pbr::pbr_functions`** - PBR lighting and material functions

**`bevy_pbr::utils`** - Utility functions for common operations

**`bevy_pbr::lighting`** - Light calculations

**`bevy_pbr::shadows`** - Shadow sampling and calculations

Import only what you need to keep shader compilation fast.

## 2D Materials

The `Material2d` trait provides equivalent functionality for 2D rendering:

```rust
#[derive(AsBindGroup, TypeUuid, Clone)]
#[uuid = "a1234567-89ab-cdef-0123-456789abcdef"]
pub struct SpriteMaterial {
    #[uniform(0)]
    tint: Color,
}

impl Material2d for SpriteMaterial {
    fn fragment_shader() -> ShaderRef {
        "shaders/sprite_material.wgsl".into()
    }
}
```

Use with `MaterialMesh2dBundle` for 2D entities. The API mirrors 3D materials.

## Shader Pipeline Specialization

Materials automatically specialize based on mesh vertex layouts and render conditions. A shader using vertex colors only compiles that code path when rendering meshes with color attributes.

The material system handles specialization automatically. Define your shader with all possible features, and Bevy generates optimized variants as needed.

## Built-in Materials

Bevy's built-in materials use the same material system:

**StandardMaterial** - PBR material for 3D rendering
**ColorMaterial** - Simple colored material for 2D/3D

These demonstrate best practices for material implementation. Check their source code for examples of advanced material features.

## Advanced: Custom Vertex Shaders

Override the vertex shader for custom vertex processing:

```rust
impl Material for MyMaterial {
    fn vertex_shader() -> ShaderRef {
        "shaders/my_vertex.wgsl".into()
    }
    
    fn fragment_shader() -> ShaderRef {
        "shaders/my_fragment.wgsl".into()
    }
}
```

Custom vertex shaders enable effects like vertex displacement, custom animation, or procedural geometry.

## Performance Considerations

**Batch by material** - Entities sharing materials render together efficiently. Minimize material variety per frame.

**Minimize bind group changes** - Switching materials incurs cost. Group draws by material when possible.

**Use texture arrays** - Multiple textures in one array reduce binding overhead compared to many individual textures.

**Specialize wisely** - Each unique combination of shader features creates a new pipeline. Keep feature variations reasonable.

## Common Patterns

### Animated Materials

Update material uniforms for animation:

```rust
fn animate_material(
    time: Res<Time>,
    mut materials: ResMut<Assets<MyMaterial>>
) {
    for (_, material) in materials.iter_mut() {
        material.time = time.elapsed_seconds();
    }
}
```

### Material Instances

Create variations of a material by adding different instances:

```rust
let red = materials.add(MyMaterial { color: Color::RED });
let blue = materials.add(MyMaterial { color: Color::BLUE });
let green = materials.add(MyMaterial { color: Color::GREEN });
```

Each instance can have different parameters while sharing shader code.

### Transparent Materials

Configure alpha blending for transparency:

```rust
impl Material for MyMaterial {
    fn alpha_mode(&self) -> AlphaMode {
        AlphaMode::Blend
    }
}
```

Transparent objects render after opaque objects with proper blending.

## WGSL Shader Syntax

Bevy uses WGSL (WebGPU Shading Language) for shaders. Key syntax elements:

**Variables:** `var`, `let`, `const`
**Types:** `f32`, `i32`, `u32`, `vec2<f32>`, `vec3<f32>`, `vec4<f32>`, `mat4x4<f32>`
**Functions:** `fn name(param: type) -> return_type { }`
**Attributes:** `@location`, `@group`, `@binding`, `@builtin`

WGSL is designed for safety and portability across platforms. The syntax is similar to Rust with explicit type annotations.

## Best Practices

**Start simple** - Begin with basic uniforms and textures. Add complexity as needed.

**Import built-ins** - Reuse Bevy's shader functions rather than reimplementing common operations.

**Test incrementally** - Make small changes and test frequently. Shader errors can be cryptic.

**Profile rendering** - Measure GPU performance. Shader optimization matters for complex materials.

**Document parameters** - Comment what each material field does. Future you will thank you.

**Version your UUIDs** - Change the TypeUuid when making breaking changes to material structure.

The material system balances ease of use with power. Simple materials need minimal code, while complex effects remain achievable. Understanding materials opens up the full creative potential of Bevy's rendering.

