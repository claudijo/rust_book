# Web Platform Support

**Added in Bevy 0.2, Expanded in Bevy 0.4**

Bevy now runs on the web with full rendering support!

## Bevy 0.2: Initial Support

Initial WASM support provided:
- Bevy ECS schedules
- Input events
- Canvas creation
- Single-threaded task scheduling

## Bevy 0.4: WebGL2 Rendering

**Added in Bevy 0.4**

Bevy now has a complete WebGL2 render backend! You can run Bevy games in the browser with full 2D and 3D rendering.

### What Works

- ✅ ECS schedules
- ✅ Input events (keyboard, mouse, touch)
- ✅ **2D and 3D rendering** (WebGL2)
- ✅ Asset loading via HTTP
- ✅ UI rendering
- ✅ Text rendering
- ✅ Sprite rendering
- ✅ PBR materials

### Current Limitations

- ⚠️ Multi-threading limited (single-threaded task scheduler)
- ⚠️ Audio support limited

## Running Bevy on Web

Check out the Bevy WebGL2 Showcase website which demonstrates various Bevy examples and games running in the browser.

### Building for Web

```bash
# Add WASM target
rustup target add wasm32-unknown-unknown

# Build your game for web
cargo build --target wasm32-unknown-unknown --release

# Serve and test locally
# (use a tool like basic-http-server or python -m http.server)
```

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

