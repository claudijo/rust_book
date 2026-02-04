# Assets

Bevy Assets are just typed data that can be referenced using asset Handles. For example, 3d meshes, textures, fonts, materials, scenes, and sounds are assets. `Assets<T>` is a generic collection of assets of type T.

## Asset Creation

```rust
fn create_texture_system(mut textures: ResMut<Assets<Texture>>) {
    // creates a new Texture asset and returns a handle, which can then be used
    // to retrieve the actual asset
    let texture_handle: Handle<Texture> = textures.add(Texture::default());
}
```

## Asset Access

```rust
fn read_texture_system(textures: Res<Assets<Texture>>, texture_handle: &Handle<Texture>) {
    // retrieves a Texture using the current entity's Handle<Texture> component
    let texture: &Texture = textures.get(texture_handle).unwrap();
}
```

## Asset Events

The `Assets<T>` collection is basically just a map from `Handle<T>` to T that records created, modified, and removed Events. These events can also be consumed as a system resource, just like any other Events:

```rust
fn system(mut state: Local<State>, texture_events: Res<Events<AssetEvent>>) {
    for event in state.reader.iter(&texture_events) {
        if let AssetEvent::Created { handle } = event {
            /* do something with created asset */
        }
    }
}
```

## Asset Server

The `Assets<T>` collection doesn't know anything about filesystems or multi-threading. This is the responsibility of the AssetServer resource:

**Bevy 0.1-0.2:**
```rust
fn system(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut textures: ResMut<Assets<Texture>>
) {
    // Had to unwrap the result
    let texture_handle: Handle<Texture> = asset_server.load("texture.png").unwrap();
    
    // Could block with load_sync (removed in 0.3)
    let sprite: &Texture = asset_server
        .load_sync(&mut textures, "sprite.png")
        .unwrap();
}
```

**Bevy 0.3+ (improved):**
```rust
fn system(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut textures: ResMut<Assets<Texture>>
) {
    // No more unwrap needed! Returns a strong handle
    let texture_handle: Handle<Texture> = asset_server.load("texture.png");

    // The texture may not be loaded yet, but you can still use the handle
    // Bevy systems will wait for assets to be ready when needed
    commands.spawn(SpriteComponents {
        material: materials.add(texture_handle.into()),
        ..Default::default()
    });

    // You can load entire folders recursively
    asset_server.load_folder("assets").unwrap();

    // Get handle for any asset (loading or loaded)
    let music: Handle<AudioSource> = asset_server
        .get_handle("assets/music.mp3")
        .unwrap();

    // Check if an asset is loaded
    if let Some(texture) = textures.get(&texture_handle) {
        // do something with texture
    }
}
```

## Asset System Improvements in Bevy 0.3

**Changed in Bevy 0.3**

The asset system underwent major improvements:

### Handle Reference Counting

Assets are now automatically freed when their handle reference count reaches zero:

```rust
// Calling load() returns a strong handle
let handle = asset_server.load("sprite.png");

// Cloning increases the reference count
let second_handle = handle.clone();

// When all handles are dropped, the asset is freed
```

### Multi-Asset Loaders

AssetLoaders can now load multiple assets of multiple types. Previously they could only produce a single asset of a single type. This is crucial for formats like GLTF that contain meshes, textures, and scenes.

### Sub-Asset Loading

You can now load specific assets from within a file:

```rust
// Load a specific mesh primitive from a GLTF file
let mesh = asset_server.load("my_scene.gltf#Mesh0/Primitive0");
```

### AssetIo Trait

The AssetServer is now backed by the AssetIo trait, allowing different storage backends:
- **Desktop**: Loads from filesystem
- **Android**: Uses Android Asset Manager  
- **Web**: Makes HTTP fetch() requests

### Asset Dependencies

Assets can depend on other assets, which are automatically loaded. For example, a scene can reference textures and meshes, and they'll all load automatically.

### Removed load_sync()

`AssetServer::load_sync()` was removed. It wasn't WASM-friendly, encouraged blocking (causing hitching), and was incompatible with the new AssetLoader API. Asset loading is now always asynchronous. Load assets, check status in systems, and change state accordingly.

## Hot Reloading

You can enable asset change detection by calling:

```rust
asset_server.watch_for_changes().unwrap();
```

This will load new versions of assets whenever their files have changed.

**Bevy 0.2 improvements:**
- Fixed deadlock in hot asset reloading
- Fixed hot asset reloading on Windows
- Fixed missing asset info path for synchronous loading

## Asset Path Loading

**Added in Bevy 0.2**

Assets are now loaded from the root path when loading directly, and the system checks for asset path existence before attempting to load.

## Asset Modification Events

**Added in Bevy 0.2**

An `AssetEvent` is now sent when modifying assets using `get_id_mut` (renamed to `get_with_id_mut` in 0.2):

```rust
// Bevy 0.2
let handle = assets.get_with_id_mut(id);
// This now triggers an AssetEvent
```

## Adding New Asset Types

To add a new asset type, implement the AssetLoader trait. This tells Bevy what file formats to look for and how to translate the file bytes into the given asset type.

Once you have implemented `AssetLoader<MyAsset>` for MyAssetLoader you can register your new loader like this:

```rust
app.add_asset_loader::<MyAsset, MyAssetLoader>();
```

Then you can access the `Assets<MyAsset>` resource, listen for change events, and call `asset_server.load("something.my_asset")`.

