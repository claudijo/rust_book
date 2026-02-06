# Assets

Games need external resources: textures, 3D models, sounds, fonts, configuration files. Bevy's asset system loads these resources efficiently, handles dependencies, and manages their lifecycle automatically.

## Understanding Assets

An asset is any external data your game loads at runtime. Bevy represents assets as strongly-typed Rust values. A `Texture` is an asset. A `Mesh` is an asset. An `AudioSource` is an asset.

Rather than passing assets around directly, you work with handles - lightweight references that point to assets:

```rust
fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    let texture_handle: Handle<Image> = asset_server.load("sprite.png");
    
    commands.spawn(SpriteBundle {
        texture: texture_handle,
        ..Default::default()
    });
}
```

The `AssetServer` loads the texture file asynchronously in the background. Your systems can use the handle immediately, and rendering waits until the asset is ready.

## The AssetServer

The `AssetServer` resource handles all asset loading. It knows how to find files, load them asynchronously, and convert them into the appropriate types:

```rust
fn load_assets(asset_server: Res<AssetServer>) {
    // Load individual assets
    let texture = asset_server.load("player.png");
    let sound = asset_server.load("jump.ogg");
    let font = asset_server.load("FiraSans-Bold.ttf");
    let mesh = asset_server.load("spaceship.gltf#Mesh0/Primitive0");
}
```

Asset paths are relative to the `assets` folder in your project. When you load `"player.png"`, Bevy looks for `assets/player.png`.

## Asset Collections

While `AssetServer` loads assets, the `Assets<T>` resource stores them. This generic collection holds all loaded assets of a given type:

```rust
fn use_texture(
    texture_handle: &Handle<Image>,
    images: Res<Assets<Image>>
) {
    if let Some(texture) = images.get(texture_handle) {
        // Access the actual texture data
        println!("Texture size: {}x{}", texture.width(), texture.height());
    }
}
```

Most of the time you don't need direct access to `Assets<T>`. The renderer, audio system, and other subsystems handle it automatically. But when you need to inspect or modify asset data at runtime, the collection is available.

## Handle Management

Handles use reference counting. When you clone a handle, the reference count increases. When all handles to an asset are dropped, Bevy automatically unloads the asset:

```rust
fn spawn_enemies(mut commands: Commands, asset_server: Res<AssetServer>) {
    let texture = asset_server.load("enemy.png");
    
    // Spawn 10 enemies using the same texture
    for _ in 0..10 {
        commands.spawn(SpriteBundle {
            texture: texture.clone(),  // Clone the handle, not the texture
            ..Default::default()
        });
    }
    
    // When all enemies despawn, the texture unloads automatically
}
```

This automatic memory management prevents resource leaks and keeps memory usage under control.

## Sub-Assets

Some file formats contain multiple assets. A GLTF file might have several meshes, materials, and textures. Bevy lets you load specific sub-assets using labels:

```rust
fn load_gltf_parts(asset_server: Res<AssetServer>) {
    // Load specific mesh from a GLTF file
    let mesh = asset_server.load("model.gltf#Mesh0/Primitive0");
    
    // Load a specific texture
    let texture = asset_server.load("model.gltf#Texture0");
    
    // Or load the entire scene
    let scene = asset_server.load("model.gltf#Scene0");
}
```

The syntax `filename#label` accesses named sub-assets within a file.

## Asset Dependencies

Assets often depend on other assets. A material references textures. A scene references meshes and materials. Bevy handles these dependencies automatically:

```rust
// Loading a scene automatically loads all its dependencies
let scene = asset_server.load("level.gltf#Scene0");

// All meshes, materials, and textures are loaded too
// You don't need to manually load each one
```

When the scene loads, Bevy's asset system ensures all dependencies load first. When you unload the scene, dependencies unload if nothing else needs them.

## Checking Load Status

Assets load asynchronously. Sometimes you need to know when loading completes:

```rust
fn check_loading(
    asset_server: Res<AssetServer>,
    texture_handle: Res<Handle<Image>>
) {
    match asset_server.get_load_state(&texture_handle) {
        LoadState::Loaded => println!("Texture ready!"),
        LoadState::Loading => println!("Still loading..."),
        LoadState::Failed => println!("Load failed!"),
        LoadState::NotLoaded => println!("Not started yet"),
    }
}
```

A common pattern is transitioning from a loading screen to gameplay once assets finish:

```rust
fn check_assets_loaded(
    mut state: ResMut<State<GameState>>,
    asset_server: Res<AssetServer>,
    assets: Res<GameAssets>
) {
    if asset_server.get_load_state(&assets.player_texture) == LoadState::Loaded
        && asset_server.get_load_state(&assets.enemy_texture) == LoadState::Loaded
    {
        state.set(GameState::Playing).unwrap();
    }
}
```

## Asset Events

The asset system emits events when assets load, change, or unload. You can react to these events in your systems:

```rust
fn handle_texture_events(
    mut events: EventReader<AssetEvent<Image>>,
    images: Res<Assets<Image>>
) {
    for event in events.iter() {
        match event {
            AssetEvent::Created { handle } => {
                println!("Texture loaded: {:?}", handle);
            }
            AssetEvent::Modified { handle } => {
                println!("Texture modified: {:?}", handle);
            }
            AssetEvent::Removed { handle } => {
                println!("Texture unloaded: {:?}", handle);
            }
        }
    }
}
```

This is particularly useful for hot reloading during development.

## Hot Reloading

During development, you often want to see asset changes without restarting the game. Bevy supports hot reloading of assets:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_startup_system(setup)
        .run();
}

fn setup(asset_server: Res<AssetServer>) {
    // Enable watching for asset changes
    asset_server.watch_for_changes().unwrap();
}
```

With hot reloading enabled, modifying a texture or sound file automatically reloads it in your running game. This dramatically speeds up iteration.

## Creating Assets at Runtime

While most assets come from files, you can create them programmatically:

```rust
fn create_texture(mut images: ResMut<Assets<Image>>) {
    // Create a solid color texture
    let mut image = Image::new_fill(
        Extent3d {
            width: 64,
            height: 64,
            depth_or_array_layers: 1,
        },
        TextureDimension::D2,
        &[255, 0, 0, 255],  // Red
        TextureFormat::Rgba8UnormSrgb,
    );
    
    let handle = images.add(image);
    // Use this handle just like a loaded asset
}
```

This is useful for procedurally generated content, debug visualizations, or runtime effects.

## Custom Asset Types

Bevy supports common asset types out of the box, but you can add your own:

```rust
#[derive(TypeUuid)]
#[uuid = "413be529-bfeb-41b3-9db0-4b8b380a2c46"]
struct LevelData {
    enemies: Vec<EnemySpawn>,
    items: Vec<ItemSpawn>,
}

impl AssetLoader for LevelDataLoader {
    fn load<'a>(
        &'a self,
        bytes: &'a [u8],
        load_context: &'a mut LoadContext,
    ) -> BoxedFuture<'a, Result<(), anyhow::Error>> {
        Box::pin(async move {
            let level_data = parse_level_file(bytes)?;
            load_context.set_default_asset(LoadedAsset::new(level_data));
            Ok(())
        })
    }

    fn extensions(&self) -> &[&str] {
        &["level"]
    }
}

// Register in your app
app.add_asset::<LevelData>()
   .add_asset_loader(LevelDataLoader);
```

Once registered, you can load your custom assets just like built-in ones:

```rust
let level = asset_server.load("level1.level");
```

## Platform-Specific Loading

Bevy's asset system adapts to different platforms automatically through the `AssetIo` trait:

**Desktop** - Loads from the filesystem
**Web** - Uses HTTP fetch requests
**Android** - Reads from the Android asset manager
**iOS** - Loads from the app bundle

You write the same code, and Bevy handles platform differences transparently.

## Compressed Textures

Large scenes contain hundreds of textures consuming gigabytes of memory. Compressed GPU textures dramatically reduce load times and memory usage while improving performance.

### Why Compress Textures?

Traditional image formats like PNG compress for storage but must decompress before the GPU uses them. This decompression is slow and the resulting textures consume large amounts of RAM. GPU-native compressed textures remain compressed in memory and decompress on-the-fly during rendering.

Benefits:

**Faster loading** - Compressed textures load 5-10x faster than PNG. A scene taking 13 seconds to load with PNG might load in 1.5 seconds with compressed textures.

**Less memory** - Compressed textures use 50-80% less RAM. More textures fit in memory simultaneously.

**Better performance** - GPUs read less data per texture access, improving frame rates. Bandwidth-limited scenes can see 10%+ performance gains.

**Mipmaps included** - Compressed texture formats support mipmaps, providing smoother, less noisy rendering at distance.

### Supported Formats

Bevy supports multiple compressed texture formats:

**.ktx2** - Khronos Texture format, widely supported
**.dds** - DirectDraw Surface, common in game development
**.basis** - Universal format that transcodes at runtime

Enable these formats with Cargo features:

```toml
[dependencies.bevy]
version = "0.7"
features = ["ktx2", "dds", "basis-universal"]
```

### Using Compressed Textures

Load compressed textures like any other asset:

```rust
fn load_textures(asset_server: Res<AssetServer>) {
    let albedo = asset_server.load("textures/wall.ktx2");
    let normal = asset_server.load("textures/wall_normal.ktx2");
    let metallic_roughness = asset_server.load("textures/wall_mr.ktx2");
}
```

The asset server detects the format automatically. Your material code doesn't change:

```rust
let material = materials.add(StandardMaterial {
    base_color_texture: Some(albedo),
    normal_map_texture: Some(normal),
    metallic_roughness_texture: Some(metallic_roughness),
    ..Default::default()
});
```

### Format Support

Different platforms support different compression formats:

**ASTC** - Mobile devices (iOS, Android), newer desktop GPUs
**BCn (BC1-BC7)** - Desktop, Xbox, PlayStation
**ETC2** - Mobile devices, OpenGL ES 3.0+

Universal formats like ETC1S and UASTC transcode to platform-specific formats at load time, providing compatibility across all platforms with a single asset file.

### Creating Compressed Textures

Use texture compression tools to create compressed assets:

**basisu** - Basis Universal compressor, creates .basis files
**ktxtools** - Create .ktx2 from various sources
**texconv** - Microsoft's texture conversion tool for .dds

Example with basisu:

```bash
basisu -mipmap -ktx2 input.png -output_file output.ktx2
```

This creates a compressed KTX2 file with mipmaps from a PNG source.

### Texture Filtering

Bevy defaults to linear texture filtering, which produces smooth results by interpolating between texture pixels. This works well for most textures and provides better visual quality than nearest-neighbor filtering.

If you need pixel-perfect rendering (like pixel art games), configure nearest filtering:

```rust
fn main() {
    App::new()
        .insert_resource(ImageSettings::default_nearest())
        .add_plugins(DefaultPlugins)
        .run();
}
```

You can also set filtering per-texture:

```rust
fn setup_texture(mut images: ResMut<Assets<Image>>, handle: Handle<Image>) {
    if let Some(image) = images.get_mut(&handle) {
        image.sampler_descriptor = ImageSampler::Descriptor(SamplerDescriptor {
            mag_filter: FilterMode::Nearest,
            min_filter: FilterMode::Nearest,
            ..Default::default()
        });
    }
}
```

Linear filtering is generally preferred as it produces smoother visuals, especially with mipmaps.

### When to Use Compression

**Large scenes** - Many textures benefit greatly from compression.

**Memory-constrained platforms** - Mobile devices, web, older hardware.

**Bandwidth-limited rendering** - Complex scenes with many texture samples per frame.

**Distribution** - Smaller downloads when shipping games.

**Don't compress** when texture quality is critical and you have memory/bandwidth to spare, or when working with procedurally generated textures that don't benefit from compression.

### Performance Considerations

Compressed textures are almost always faster than uncompressed:
- Faster loading from disk
- Less memory bandwidth during rendering  
- More textures cached in GPU memory
- Mipmaps improve rendering quality and speed

The only cost is initial compression time during asset creation, which is a one-time offline process.

## Best Practices

**Load assets early** - Start loading during a loading screen or splash screen. Don't wait until you need an asset to start loading it.

**Use strong handles** - Keep handles in resources or components where they'll stay alive. Don't rely on weak handles for assets you actively use.

**Batch loads** - Load multiple assets together rather than in separate frames to improve load times.

**Handle load failures** - Check load states for critical assets. Don't assume loads always succeed.

**Use hot reloading** - Enable it during development to iterate faster on art and design.

Bevy's asset system handles the complexity of loading, storing, and managing external resources while providing a simple API. Understanding handles, the `AssetServer`, and asset collections will serve you well across all types of games.

