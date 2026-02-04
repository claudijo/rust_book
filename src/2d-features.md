# 2D Features

Bevy provides several features for 2D game development.

## Sprites

**Bevy 0.6+ (simplified):**
```rust
let texture = asset_server.load("icon.png");
commands.spawn_bundle(SpriteBundle {
    texture,
    sprite: Sprite {
        color: Color::rgb(0.8, 0.2, 0.2),  // Tint color
        custom_size: Some(Vec2::new(256.0, 256.0)),
        ..Default::default()
    },
    ..Default::default()
});
```

**Before 0.6:**
```rust
let texture = asset_server.load("icon.png").unwrap();
commands.spawn_bundle(SpriteBundle {
    material: materials.add(ColorMaterial {
        texture: Some(texture),
        color: Color::WHITE,
    }),
    sprite: Sprite {
        size: Vec2::new(256.0, 256.0),
        resize_mode: SpriteResizeMode::Manual,
        ..Default::default()
    },
    ..Default::default()
});
```

**Changes in 0.6:**
- Texture is now directly on SpriteBundle (no more ColorMaterial)
- Color is on the Sprite component
- `custom_size` replaces `size` + `resize_mode`
- Much simpler and more intuitive!

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

## Bevy 0.2 Improvements

### Sprite Resize Mode

`SpriteResizeMode` allows choosing how sprite resizing should be handled. Automatic by default.

```rust
// Control how sprites resize
sprite.resize_mode = SpriteResizeMode::Manual;
```

### Fixed Sprite Clipping

Transparent sprites no longer clip incorrectly at the same depth. Sprites at the same depth now render properly.

