# Version Tracking

This document tracks which content came from which Bevy version.

## Bevy 0.1 (August 10, 2020)

**Source**: https://bevyengine.org/news/introducing-bevy/

**Status**: ✅ Complete - All content integrated

### Content Added

#### Introduction
- Design goals (Capable, Simple, Data Focused, Modular, Fast, Productive)
- Key features overview
- General purpose features list
- Current state (prototyping phase)

#### Bevy Apps
- Simplest app example
- Default plugins
- Manual plugin registration
- Modularity philosophy

#### Bevy ECS
- ECS pattern explanation
- Components as normal structs
- Systems as normal functions
- Entities as unique IDs
- Complete ergonomics example
- Performance benchmarks (ecs_bench)
- System types:
  - For-each systems
  - Query systems
  - Change detection (Added, Mutated, Changed, Removed)
  - Multiple queries
  - Entity queries and direct component access
  - Resources (Res, ResMut)
  - Local resources
  - Empty systems
  - With/Without filters
  - Thread-local systems
- Stages
- Commands
- How function systems work (.system() conversion)
- Built on Hecs (forked with additions)

#### Bevy UI
- Design philosophy (fully embracing Bevy way)
- ECS integration benefits
- Building blocks (Node, Transform, Style)
- Layout with Stretch (flexbox)
- Relative positioning
- Absolute positioning
- Parenting
- Flexbox centering example
- Text components with fonts
- Image components
- Interaction events (Clicked, Hovered, None)

#### 2D Features
- Sprites from textures
- Sprite sheets / texture atlases
- Dynamic texture atlas generation

#### 3D Features
- GLTF model loading (as meshes)
- Depth-based draw order (front-to-back, back-to-front)
- Parenting with transform propagation
- MSAA (Multi-Sample Anti-Aliasing)

#### Scenes
- Scene composition concept
- File format (RON, flat list of entities/components)
- Entity IDs (optional, random if not provided)
- Loading (preserves IDs) vs Instancing (new IDs)
- SceneSpawner resource
- Saving worlds to scenes
- Hot scene reloading
- Built on Properties and Assets

#### Properties
- Runtime reflection for Rust
- set_prop_val / prop_val by string name
- Dynamic property setting
- DynamicProperties (schemaless)
- Property patching (apply)
- to_dynamic conversion
- Serialization via serde
- Use cases (scenes, editor undo/redo, property animation)
- Derive Properties requirement

#### Events
- Double-buffered event system
- add_event registration
- Events resource
- EventReader for consumption
- Zero-allocation consumers
- Complete example (MyEvent)
- Built-in events (window, assets, input)
- One-chance-per-update trade-off

#### Assets
- Asset handles (Handle<T>)
- Assets<T> collection
- Asset creation (add)
- Asset access (get)
- Asset events (Created, Modified, Removed)
- AssetServer for filesystem/threading
- Async loading
- load_asset_folder
- get_handle
- load_sync for immediate access
- Hot reloading (watch_for_changes)
- Custom asset types (AssetLoader trait)

#### Sound
- Audio loading via AssetServer
- AudioOutput resource
- play (async) and play_source (immediate)
- Future plans for more features

#### Render Graph
- Atomic units of render logic as nodes
- Node dependencies create graph edges
- Parallel rendering analysis
- Built-in nodes (CameraNode, PassNode, RenderResourcesNode, etc.)
- Built-in subgraphs (2D, 3D, UI)
- Extensibility
- Data-driven shaders (RenderResources derive)
- add_system_node and add_node_edge
- Shader uniforms binding
- Shader defs (conditional compilation per-entity)
- ShaderDefs derive
- Shader layout reflection (SpirV)

#### Productive Compile Times
- Productivity as design goal
- Acceptability scale (0-1s ideal, 10+s unusable)
- Iterative vs clean compiles
- Current performance (~0.8-3s with fast compiles)
- Why Rust compiles slowly:
  - Generic monomorphization
  - Link time
  - LLVM
- Game engine challenges (many domains, dependencies)
- Rust challenges (static linking, slow default linker, easy dependencies)
- Bevy's pragmatic approach (willing to take dependencies)
- Fast compiles configuration:
  - LLD linker
  - Nightly Rust compiler
  - Generic sharing
- Platform limitations (MacOS, Windows)
- Future improvements (dynamic linking, Cranelift)

#### What's Next
- Physically Based Rendering (PBR, shadows, lighting, skeletal animation, GLTF improvements, AO)
- Editor (built as Bevy App, uses UI/ECS/Scene/Properties)
- Platform support (Android, iOS, Web via winit/wgpu)
- Render batching and instancing
- Canvas (immediate-mode drawing API)
- Animation (code-first, then property-based timeline)
- Nicer scene format (indented hierarchies, nested scenes)
- Dynamic plugin loading (TypeId fix needed)
- Physics (pluggable interface, nphysics/ncollide)
- Polish (render APIs, materials)
- Documentation (proportional to stability)

### Chapter Mapping

| Chapter | Primary Source Section(s) |
|---------|---------------------------|
| Introduction | Intro, Design Goals, Features List |
| Bevy Apps | Bevy Apps section |
| Bevy ECS | Bevy ECS section (Ergonomics, Performance, Features, Good Bones) |
| Bevy UI | Bevy UI section (Building Blocks through Interaction Events) |
| 2D Features | 2D Features section |
| 3D Features | 3D Features section |
| Scenes | Scenes section |
| Properties | Properties section |
| Events | Events section |
| Assets | Assets section |
| Sound | Sound section |
| Render Graph | Render Graph section (including Data Driven Shaders, Shader Defs) |
| Productive Compile Times | Productive Compile Times section |
| What's Next | What's Next section |

---

## Bevy 0.2 (September 19, 2020)

**Source**: https://bevyengine.org/news/bevy-0-2/

**Status**: ✅ Complete - All content integrated

### New Chapters Added

1. **Task System** - Custom async task system, CPU improvements, context-specific pools
2. **Web Platform Support** - Initial WASM support, limitations noted
3. **Transforms** - Complete rewrite from separate components to unified Transform
4. **Input** - Gamepad support, new iterator methods

### Major Updates to Existing Chapters

1. **Bevy ECS** - Parallel queries, Or queries, performance improvements (generational IDs, read-only queries, removed locking)
2. **Assets** - Hot reloading fixes, asset events, path checking
3. **Bevy UI** - Multiline text, default size change, cloning support
4. **2D/3D Features** - Sprite resize mode, GLTF improvements
5. **Sound** - Custom rodio source, Decodable support
6. **Scenes** - Unload methods
7. **Render Graph** - Resource tracking fixes, iOS improvements
8. **Bevy Apps** - Startup stage control

---

## Bevy 0.3 - 0.18

**Status**: ⏳ Awaiting input

Will be added when release notes are provided.

### Integration Plan

When new versions are added:
1. Read complete release notes
2. Identify new features
3. Identify changed features
4. Identify deprecated features
5. Update existing chapters with changes
6. Add new chapters for new major features
7. Document version in this file
8. Keep structure lean (avoid over-fragmentation)

---

## Version History Summary

- **Bevy 0.1**: Foundation complete (14 chapters, ~6,100 words)
- **Bevy 0.2**: Integrated (18 chapters, ~9,000+ words, 4 new, 9 updated)
- **Bevy 0.3+**: Pending

