# What's Next?

These will be the focus areas over the coming months:

## Physically Based Rendering (PBR)

Bevy's current 3D renderer is extremely bare bones. Improving the 3D renderer is a priority. Planned additions include:
- PBR shaders
- Shadows
- More lighting options
- Skeletal animation
- Improved GLTF import
- Ambient occlusion (implementation TBD)

## Editor

Bevy was architected with a visual editor in mind. The Scene and Properties systems were specifically built to make game↔editor dataflow nicer. The editor will be built as a Bevy App and will utilize the existing Bevy UI, Bevy ECS, Scene, and Properties features. 

The "build the editor in the engine" approach means improvements to the editor are often improvements to the engine, and vice-versa. It also ensures that Bevy is capable of building non-game apps and tooling.

## Platform Support: Android, iOS, Web

Under the hood Bevy uses winit (for multi-platform windows and input) and wgpu (for multi-platform rendering). Each of these projects has varying degrees of support for these platforms. Bevy was designed to be platform-agnostic, so supporting these platforms should be possible with a little bit of work.

## Render Batching and Instancing

Right now Bevy can render plenty fast for most use cases, but when it comes to rendering huge amounts of objects (tens of thousands) it isn't quite there yet. To accomplish that, batching/instancing will be implemented. The general gist is grouping as much geometry and data into the smallest number of draw calls possible, while also reducing GPU state changes as much as possible. Bevy's data driven shader approach should make the instancing implementation simple and extensible.

## Canvas

Right now the only way to draw UI and 2D scenes is via sprites and rectangles. Bevy needs an immediate-mode drawing API capable of drawing anti-aliased curves and shapes. This can then be used for code-driven drawing of things like rounded corners in Bevy UI, performance graphs in the editor, etc. Projects like pathfinder or lyon may be integrated for this.

## Animation

Animation permeates almost everything in gamedev. First, a general purpose code-first animation system will be added. Then on top of that, a property-based timeline system that can be saved to a config file and visualized/edited in the Bevy Editor.

## Nicer Scene Format

The current scene format is workable, but it isn't yet ideal for manual scene composition because it is a flat list of unordered entities. Nested scenes will be added and the format will be improved.

## Dynamic Plugin Loading

To mitigate the cost of compiling and linking plugins and make hot-code-reloading possible, dynamic App plugin loading will be provided. Bevy actually already supports this feature, but there is one hangup: Rust's TypeId. TypeIds are unstable across binaries, which means TypeId::of::<T>() in the host binary will not match TypeId::of::<T>() in a dynamically loaded binary. Since Bevy ECS uses TypeIds, dynamically loaded ECS types won't behave correctly. The fix is to apply the same approach used in Legion ECS to Bevy ECS.

## Physics

Many games require collision detection and physics. A plug-able physics interface with nphysics/ncollide as the first backend is planned.

## Polish

There are plenty of areas that need more design work or features. The core Render Graph is in pretty good state, but the mid-level and high level rendering APIs need more time and experimentation. Material composition also needs to be rethought. Check out the Bevy issue tracker on GitHub for all improvements on the radar.

## Documentation

Bevy's APIs are still quite unstable, so documentation is spotty. The Bevy Book only covers relatively mature areas of the engine and the Rust API Docs have plenty of gaps. The philosophy is "documentation proportional to stability". As features stabilize and design patterns emerge, documentation efforts will increase.

