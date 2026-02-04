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

```rust
fn system(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut textures: ResMut<Assets<Texture>>
) {
    // this will begin asynchronously loading "texture.png" in parallel
    let texture_handle: Handle<Texture> = asset_server.load("texture.png").unwrap();

    // the texture may not be loaded yet, but you can still add the handle as a
    // component immediately. Whenever possible, internal Bevy systems will wait
    // for assets to be ready before using them:
    let entity = commands.spawn((texture_handle,));

    // you can also asynchronously load entire folders (recursively) by adding
    // them as an "asset folder"
    asset_server.load_asset_folder("assets").unwrap();

    // you can get the handle of any asset (either currently loading or loaded) like this:
    let music_handle: Handle<AudioSource> = asset_server
        .get_handle("assets/music.mp3")
        .unwrap();

    // when assets have finished loading, they are automatically added to the
    // appropriate Assets<T> collection. You can check if an asset is ready like this:
    if let Some(texture) = textures.get(&texture_handle) {
        // do something with texture
    }

    // sometimes you want access to an asset immediately. You can block the current
    // system until an asset has finished loading and immediately update Assets<T>
    // using the "load_sync" method
    let cool_sprite: &Texture = asset_server
        .load_sync(&mut textures, "assets/cool_sprite.png")
        .unwrap();
}
```

## Hot Reloading

You can enable asset change detection by calling:

```rust
asset_server.watch_for_changes().unwrap();
```

This will load new versions of assets whenever their files have changed.

## Adding New Asset Types

To add a new asset type, implement the AssetLoader trait. This tells Bevy what file formats to look for and how to translate the file bytes into the given asset type.

Once you have implemented `AssetLoader<MyAsset>` for MyAssetLoader you can register your new loader like this:

```rust
app.add_asset_loader::<MyAsset, MyAssetLoader>();
```

Then you can access the `Assets<MyAsset>` resource, listen for change events, and call `asset_server.load("something.my_asset")`.

