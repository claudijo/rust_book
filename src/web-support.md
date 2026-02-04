# Web Platform Support

**Added in Bevy 0.2**

A subset of Bevy now runs on the web using WebAssembly (WASM)!

## What Works

Specifically, Bevy apps can:
- Run Bevy ECS schedules
- React to input events
- Create an empty canvas (using winit)
- Use single-threaded task scheduling
- **Load assets (added in 0.3)** - via HTTP fetch() requests

### WASM Asset Loading

**Added in Bevy 0.3**

You can now load assets in WASM just like on any other platform:

```rust
asset_server.load("sprite.png");
```

If the asset hasn't already been loaded, this makes a `fetch()` request to retrieve the asset over HTTP. The AssetIo trait (added in 0.3) provides the abstraction that enables different storage backends:
- Desktop: Filesystem
- Android: Android Asset Manager
- WASM: HTTP fetch() requests

## Current Limitations

It's important to note that there are still missing pieces:
- 2D/3D rendering (not yet supported)
- Multi-threading (not yet supported)
- Sound (not yet supported)

## First WASM Bevy Game

Despite the limitations, @mrk-its built the first WASM Bevy game: **bevy-robbo**

The game uses Bevy for game logic and works around the render limitations by passing ASCII art game state from Bevy systems to JavaScript functions for rendering.

## Try It Yourself

You can experiment with Bevy WASM examples by following the instructions in the Bevy repository.

## Implementation Details

The web support includes:
- WASM support for `bevy_winit`
- Canvas creation under WebAssembly
- Single-threaded task scheduler for WebAssembly

## Future

While the initial web support is limited, it represents a huge first step toward full web platform compatibility. Future versions will add rendering, multi-threading, and audio support.

