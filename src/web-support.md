# Web Platform

Bevy games run in web browsers through WebAssembly and WebGL. Deploy your game to reach players without downloads or installation.

## WebAssembly Compilation

Rust compiles to WebAssembly, a binary format that runs in browsers at near-native speeds. Bevy leverages this to bring full-featured games to the web.

Add the WebAssembly target:

```bash
rustup target add wasm32-unknown-unknown
```

Build your game for web:

```bash
cargo build --release --target wasm32-unknown-unknown
```

This produces a `.wasm` file that browsers can execute.

## WebGL Rendering

Browsers provide WebGL for 3D graphics. Bevy's rendering system works through WebGL2, enabling the same graphics capabilities as desktop:

- 2D and 3D rendering
- PBR materials
- Lighting and shadows
- UI and text
- Sprites and textures

Your rendering code works identically on web and desktop. Bevy handles the platform differences internally.

## Asset Loading

Web assets load over HTTP:

```rust
fn load_assets(asset_server: Res<AssetServer>) {
    let texture = asset_server.load("player.png");
    let model = asset_server.load("spaceship.gltf");
}
```

The asset server makes fetch requests when loading web builds. This is transparent - use the same API as desktop platforms.

Host assets alongside your WebAssembly file so the browser can retrieve them.

## Input Handling

Web input works through browser APIs:

```rust
fn handle_input(
    keyboard: Res<Input<KeyCode>>,
    mouse: Res<Input<MouseButton>>,
    touches: Res<Touches>
) {
    if keyboard.pressed(KeyCode::Space) {
        // Handle keyboard
    }
    
    if mouse.just_pressed(MouseButton::Left) {
        // Handle mouse
    }
    
    for touch in touches.iter() {
        // Handle touch on mobile browsers
    }
}
```

Keyboard, mouse, and touch events all work normally. The same input code runs on desktop, mobile, and web.

## HTML Integration

Embed your game in a web page:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Bevy Game</title>
    <style>
        canvas {
            width: 100%;
            height: 100%;
        }
    </style>
</head>
<body>
    <canvas id="bevy-canvas"></canvas>
    <script type="module">
        import init from './my_game.js';
        init('./my_game_bg.wasm');
    </script>
</body>
</html>
```

The canvas element displays your game. Standard HTML/CSS styling applies.

## Performance Considerations

Web builds have unique constraints:

**Download size matters** - Users wait for your game to download. Optimize WASM size with release builds and compression.

**Browser limitations** - WebGL is less capable than native APIs. Test on target browsers and reduce complexity if needed.

**No multi-threading** - Web builds run single-threaded due to browser restrictions. Design systems accordingly.

**Memory limits** - Browsers impose memory limits. Monitor usage and unload unused assets.

## Building for Production

Optimize web builds:

```bash
# Build with optimizations
cargo build --release --target wasm32-unknown-unknown

# Further optimize with wasm-opt
wasm-opt -Oz -o output.wasm input.wasm
```

Use tools like `wasm-bindgen` and `wasm-pack` to generate JavaScript bindings and optimize deployment.

## Hosting

Host your game on any static web server:
- GitHub Pages
- Netlify
- Vercel
- Amazon S3
- Your own server

Web builds are just static files - HTML, JavaScript, and WebAssembly. No special hosting requirements.

## Browser Compatibility

Modern browsers support WebAssembly and WebGL2:
- Chrome, Edge - Excellent support
- Firefox - Excellent support
- Safari - Good support (some WebGL quirks)

Test on multiple browsers. Provide fallback messages for unsupported browsers.

## Limitations

Web platforms have inherent constraints:

**Single-threaded execution** - Browsers restrict threading. Parallel task pools don't provide the same benefits as desktop.

**Async complexity** - Everything loads asynchronously. Handle loading states appropriately.

**No filesystem access** - Save data to browser storage or cloud services, not local files.

**Performance overhead** - WebAssembly and WebGL add overhead compared to native execution.

## Best Practices

**Optimize asset sizes** - Compress textures, use efficient formats. Downloads affect load times.

**Show loading progress** - Users need feedback while assets load. Provide loading screens or progress bars.

**Test early** - Web constraints differ from desktop. Test web builds throughout development.

**Handle different screen sizes** - Web builds run on diverse devices. Design responsive UIs.

**Provide mobile support** - Many web players use mobile devices. Support touch controls.

**Monitor performance** - Profile web builds separately. Optimization priorities differ from desktop.

Web deployment makes your game accessible to anyone with a browser. Understanding platform constraints and optimizing accordingly creates smooth web experiences.

