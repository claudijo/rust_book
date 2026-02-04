# Render Graph

All render logic is built on top of Bevy's RenderGraph. The Render Graph is a way to encode atomic units of render logic. For example, you might create graph nodes for a 2D pass, UI pass, cameras, texture copies, swap chains, etc.

## How It Works

Connecting a node to another node indicates that there is a dependency of some kind between them. By encoding render logic this way, the Bevy renderer is able to analyze dependencies and render the graph in parallel. It also has the benefit of encouraging developers to write modular render logic.

## Built-in Nodes

Bevy includes a number of nodes by default:

- CameraNode
- PassNode
- RenderResourcesNode
- SharedBuffersNode
- TextureCopyNode
- WindowSwapChainNode
- WindowTextureNode

## Built-in Subgraphs

Bevy provides subgraphs for:
- 2D rendering
- 3D rendering
- UI rendering

You are welcome to create your own nodes, your own graphs, or extend the included graphs!

## Data Driven Shaders

Components and Assets can derive the RenderResources trait, which enables them to be directly copied to GPU resources and used as shader uniforms.

Binding uniforms to a custom shader is literally as simple as deriving RenderResources on your component or asset:

```rust
#[derive(RenderResources, Default)]
struct MyMaterial {
    pub color: Color,
}
```

And then adding a new RenderResourceNode to the Render Graph:

```rust
// create the new node
render_graph.add_system_node("my_material", RenderResourcesNode::<MyMaterial>::new(true));

// connect the new node to the "main pass node"
render_graph.add_node_edge("my_material", base::node::MAIN_PASS).unwrap();
```

From there, MyMaterial components will be automatically copied to GPU buffers. A shader can then reference an entity's MyMaterial like this:

```glsl
layout(set = 1, binding = 1) uniform MyMaterial_color {
    vec4 color;
};
```

## Shader Defs

Components and Assets can also add "shader defs" to selectively enable shader code on a per-entity basis:

```rust
#[derive(RenderResources, ShaderDefs, Default)]
struct MyMaterial {
    pub color: Color,
    #[render_resource(ignore)]
    #[shader_def]
    pub always_blue: bool,
}
```

Then in your fragment shader you could do something like this:

```glsl
void main() {
    o_Target = color;
# ifdef MYMATERIAL_ALWAYS_BLUE
    o_Target = vec4(0.0, 0.0, 1.0, 1.0);
# endif
}
```

Any entity with a MyMaterial component and `always_blue: true` will be rendered blue. If always_blue is false, it will be rendered with color.

We currently use this feature for toggling "shadeless" rendering and optional textures.

## Shader Layout Reflection

Bevy can automatically reflect shader data layouts from SpirV shaders (and from GLSL shaders by compiling them to SpirV). This means that creating custom shaders is as simple as this:

```rust
let shader_stages = ShaderStages {
    vertex: shaders.add(Shader::from_glsl(ShaderStage::Vertex, VERTEX_SHADER)),
    fragment: Some(shaders.add(Shader::from_glsl(ShaderStage::Fragment, FRAGMENT_SHADER))),
};
let pipeline_handle = pipelines.add(PipelineDescriptor::default_config(shader_stages));
```

## Bevy 0.2 Improvements

### Render Resources Tracking

**Fixed in Bevy 0.2**

Properly track added and removed RenderResources in RenderResourcesNode. This fixes issues where entities vanished or changed color when new entities were spawned/despawned.

### iOS Shader Compilation

**Added in Bevy 0.2**

iOS now uses shaderc-rs for GLSL to SPIR-V compilation, improving shader support on Apple platforms.

### Render Feature Flag

**Added in Bevy 0.2**

Added a `render` feature flag, which makes the entire render pipeline optional. This is useful for headless servers or when building tools that don't need rendering.

