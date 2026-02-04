# The New Bevy Renderer

**Added in Bevy 0.6**

Bevy 0.6 introduces a completely rewritten renderer that is faster, prettier, simpler, and more modular than the previous renderer.

## Why a New Renderer?

The old renderer (0.1-0.5) had strengths:
- Modular render logic via Render Graph
- Multiple backends support
- High-level data-driven API

But significant shortcomings:
- **Complex** - High abstraction overhead and invented jargon
- **Often slow** - Suboptimal performance (especially sprites)
- **User-facing internals** - Internal render state cluttered user APIs
- **Hard to repeat** - Viewports, multi-window, shadow maps required special casing

The new renderer addresses all of these issues.

## Key Improvements

### Faster

- **More parallel** - Better multi-threading
- **Less computation per-entity** - Efficient data structures
- **Better CPU→GPU dataflow** - Minimized data transfer
- **Pipelined rendering** - Overlap app logic with rendering

### Prettier

New graphical features in 0.6:
- **Directional and point light shadows**
- **Clustered forward rendering** - Draw many more lights
- **Spherical area lights**
- **Frustum culling** - Only render visible objects

Future features in development:
- Cascaded shadow maps
- Bloom
- Particles
- Shadow filters

### Simpler

- Fewer layers of abstraction
- Simpler data flow
- Better low, mid, and high-level interfaces
- Direct wgpu access when needed

### More Modular

- Standardized 2D and 3D core pipelines
- Extensible Render Phases and Views
- Composable entity/component-driven draw functions
- Shader imports
- Repeatable render pipelines via "sub graphs"

## Pipelined Rendering

The cornerstone of the new renderer is **pipelined rendering**.

### How It Works

**Traditional (non-pipelined):**
```
Frame N: [App Logic] → [Render]
Frame N+1:             [App Logic] → [Render]
```

**Pipelined:**
```
Frame N:   [App Logic] → [Render]
Frame N+1:             [App Logic] → [Render]
```

The app logic for frame N+1 runs while rendering frame N!

### Benefits

1. **Increased Parallelism** - Better CPU utilization
2. **Clearer Dataflow** - Hard separation between app and render logic
3. **Better Structure** - Fixed synchronization point (Extract step)

### The Rendering Pipeline

The new renderer has four stages:

1. **Extract** - Copy data from the app world to the render world
2. **Prepare** - Write data to GPU buffers
3. **Queue** - Set up render operations
4. **Render** - Execute draw commands

```rust
// Extract: Pull data from app world
Extract(app_world) → render_world

// Prepare: Upload to GPU
Prepare(render_world) → GPU buffers

// Queue: Build draw commands
Queue(render_world) → Draw commands

// Render: Execute on GPU
Render(draw_commands) → Screen
```

This clear separation makes the renderer easier to understand and extend.

## Render Worlds

**The App World and Render World are separate!**

- **App World** - Contains game state, runs game logic
- **Render World** - Contains only render state, runs render logic

During the **Extract** stage, relevant data is copied from App World to Render World. This separation enables:
- Clean pipelining (no data races)
- Clear ownership
- Simplified render logic

## Clustered Forward Rendering

**Added in Bevy 0.6**

Clustered forward rendering enables rendering many lights efficiently.

### How It Works

The view frustum is divided into clusters (3D grid of regions). For each cluster, we track which lights affect it. When rendering, we only consider lights in the current cluster.

### Benefits

- **Many lights** - Hundreds of lights with good performance
- **Dynamic lights** - Add/remove/move lights freely
- **No prebaking** - Fully dynamic lighting

Before clustered rendering, you were limited to a small number of lights. Now you can have complex, dynamic lighting scenarios.

## Frustum Culling

**Added in Bevy 0.6**

Objects outside the camera's view frustum are automatically culled (not rendered).

### Benefits

- **Automatic** - No configuration needed
- **Fast** - Efficient culling algorithm
- **Accurate** - Uses object bounding boxes

This significantly improves performance in large scenes where many objects are off-screen.

## Native WebGL2 Support

**Added in Bevy 0.6**

The new renderer fully supports WebGL2, enabling Bevy games to run in web browsers with complete rendering capabilities.

### What Works

- ✅ 2D and 3D rendering
- ✅ PBR materials
- ✅ Shadows
- ✅ All shader features
- ✅ Text rendering
- ✅ UI rendering

You can test Bevy examples directly in your browser!

## Custom Materials

**Added in Bevy 0.6**

High-level custom materials are now much easier:

```rust
#[derive(AsBindGroup, TypeUuid, Clone)]
#[uuid = "f690fdae-d598-45ab-8225-97e2a3f056e0"]
pub struct CustomMaterial {
    #[uniform(0)]
    color: Color,
    #[texture(1)]
    #[sampler(2)]
    color_texture: Handle<Image>,
}

impl Material for CustomMaterial {
    fn fragment_shader() -> ShaderRef {
        "shaders/custom_material.wgsl".into()
    }
}
```

Much simpler than before! The new material system automatically handles:
- Uniform binding
- Texture binding
- Render pipeline setup
- GPU resource management

## Shader Improvements

**Added in Bevy 0.6**

### WGSL Support

Bevy now supports WGSL (WebGPU Shading Language), the new standard shader language:

```wgsl
[[stage(fragment)]]
fn fragment(in: FragmentInput) -> [[location(0)]] vec4<f32> {
    return textureSample(base_color_texture, base_color_sampler, in.uv);
}
```

### Shader Imports

Shaders can now import other shaders:

```wgsl
#import bevy_pbr::mesh_view_bindings
#import bevy_pbr::utils

[[stage(fragment)]]
fn fragment() -> [[location(0)]] vec4<f32> {
    // Use imported functions
}
```

This enables:
- Code reuse across shaders
- Standardized PBR functions
- Easier maintenance

### Shader Preprocessor

Conditional compilation in shaders:

```wgsl
#ifdef VERTEX_COLORS
    out.color = vertex.color;
#else
    out.color = vec4<f32>(1.0, 1.0, 1.0, 1.0);
#endif
```

## Performance Improvements

The new renderer brings massive performance improvements:

### Sprite Rendering

Old renderer: Slow, high overhead
New renderer: 10x+ faster for sprite-heavy games

### 3D Rendering

- Better batching
- Efficient GPU data transfer
- Parallel command building
- Clustered lighting

## Render Phases

Rendering is organized into phases:

1. **Opaque** - Solid objects (front-to-back for early-z)
2. **Alpha Mask** - Alpha-tested objects
3. **Transparent** - Transparent objects (back-to-front for blending)

Each phase can have custom rendering logic. This enables:
- Correct rendering order
- Extensible rendering
- Custom render phases for special effects

## Future Work

The new renderer is a foundation for many upcoming features:

- **Cascaded shadow maps** - Better directional light shadows
- **Bloom** - Glow effects
- **Particles** - Efficient particle systems
- **Shadow filters** - Soft shadows
- **Screen space reflections** - Realistic reflections
- **Temporal anti-aliasing** - Better anti-aliasing

## Migration Notes

The new renderer comes with breaking changes:

1. **Visible → Visibility** - Component renamed
2. **is_transparent removed** - Use alpha_mode on materials
3. **Light → PointLight** - Renamed for clarity
4. **Custom materials** - New API (much simpler!)
5. **Render resources** - Different structure

See the migration guide for details on updating your code.

## Summary

The new Bevy renderer represents a massive step forward:

- ✅ **10x+ faster** in many cases
- ✅ **Prettier** with shadows, clustered lighting
- ✅ **Simpler** to use and extend
- ✅ **More modular** and composable
- ✅ **Production-ready** architecture

This foundation enables Bevy to compete with commercial engines while remaining open source and written in Rust. The renderer is battle-tested, taking inspiration from AAA engines like Destiny while incorporating best practices from the Rust graphics community.

The new renderer unlocks Bevy's potential for serious game development and sets the stage for future graphical improvements!

