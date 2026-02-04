# 2D Features

Bevy provides several features for 2D game development.

## Sprites

You can use any Texture asset as a sprite directly:

```rust
let texture = asset_server.load("icon.png").unwrap();
commands.spawn(SpriteComponents {
    material: materials.add(texture.into()),
    ..Default::default()
});
```

## Sprite Sheets

Sprite sheets (also known as texture atlases) can be used for animations, tile sets, or just for optimized sprite rendering.

```rust
let texture_atlas = TextureAtlas::from_grid(texture_handle, texture.size, 7, 1);
let texture_atlas_handle = texture_atlases.add(texture_atlas);
commands.spawn(SpriteSheetComponents {
    texture_atlas: texture_atlas_handle,
    sprite: TextureAtlasSprite::new(0),
    ..Default::default()
});
```

## Dynamic Texture Atlas Generation

Sprites are often produced as individual files. Bevy can combine them into a single sprite sheet dynamically!

```rust
for sprite_handle in sprite_handles.iter() {
    let texture = textures.get(&handle).unwrap();
    texture_atlas_builder.add_texture(handle, &texture);
}

let texture_atlas = texture_atlas_builder.finish(&mut textures).unwrap();
```

