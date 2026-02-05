# 2D Rendering

Bevy provides everything you need for 2D games: sprites, sprite sheets, animations, and efficient batch rendering.

## Sprites

Sprites are 2D images rendered in your game world. Create a sprite by spawning an entity with a SpriteBundle:

```rust
fn spawn_player(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn(SpriteBundle {
        texture: asset_server.load("player.png"),
        ..Default::default()
    });
}
```

The sprite appears at the origin (0, 0) with its original size and no tint. Customize these properties through the bundle's components:

```rust
commands.spawn(SpriteBundle {
    texture: asset_server.load("enemy.png"),
    transform: Transform::from_xyz(100.0, 50.0, 0.0),
    sprite: Sprite {
        color: Color::rgb(1.0, 0.5, 0.5),  // Reddish tint
        custom_size: Some(Vec2::new(64.0, 64.0)),
        ..Default::default()
    },
    ..Default::default()
});
```

The `Transform` component positions the sprite in 2D space (using X and Y, leaving Z for depth sorting). The `Sprite` component controls appearance: color tinting, size, and flipping.

## Sprite Colors

The `color` field tints the sprite by multiplying its pixels with the color. White (the default) shows the original image. Other colors tint it:

```rust
sprite.color = Color::rgb(0.5, 0.5, 1.0);  // Blue tint

sprite.color = Color::rgba(1.0, 1.0, 1.0, 0.5);  // Half transparent
```

This is useful for damage flashes, team colors, or fade effects.

## Custom Sprite Sizes

By default, sprites render at their texture's pixel dimensions. Override this with `custom_size`:

```rust
sprite.custom_size = Some(Vec2::new(128.0, 128.0));  // Always 128x128
```

Without `custom_size`, sprites use the texture's actual size. With it, they scale to match regardless of the source image dimensions.

## Flipping Sprites

Flip sprites horizontally or vertically without loading separate textures:

```rust
fn player_movement(
    keyboard: Res<Input<KeyCode>>,
    mut query: Query<(&mut Transform, &mut Sprite), With<Player>>
) {
    for (mut transform, mut sprite) in query.iter_mut() {
        if keyboard.pressed(KeyCode::A) {
            transform.translation.x -= 5.0;
            sprite.flip_x = true;  // Face left
        }
        if keyboard.pressed(KeyCode::D) {
            transform.translation.x += 5.0;
            sprite.flip_x = false;  // Face right
        }
    }
}
```

Flipping is efficient - no texture duplication or additional GPU memory. Perfect for character facing directions.

## Sprite Sheets

Sprite sheets pack multiple images into a single texture. This reduces draw calls and improves performance. Games use sprite sheets for animations, tile sets, and UI elements.

Define a sprite sheet by dividing a texture into a grid:

```rust
fn load_character(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut texture_atlases: ResMut<Assets<TextureAtlas>>
) {
    let texture = asset_server.load("character_sheet.png");
    
    let atlas = TextureAtlas::from_grid(
        texture,
        Vec2::new(32.0, 32.0),  // Each frame is 32x32 pixels
        8,  // 8 columns
        4,  // 4 rows
        None,
        None,
    );
    
    let atlas_handle = texture_atlases.add(atlas);
    
    commands.spawn(SpriteSheetBundle {
        texture_atlas: atlas_handle,
        sprite: TextureAtlasSprite::new(0),  // Start at frame 0
        ..Default::default()
    });
}
```

This creates a sprite sheet from an 8x4 grid, yielding 32 individual sprites numbered 0-31.

## Animating Sprites

Animate sprites by changing the sprite index over time:

```rust
#[derive(Component)]
struct AnimationTimer(Timer);

fn animate_sprite(
    time: Res<Time>,
    mut query: Query<(
        &mut AnimationTimer,
        &mut TextureAtlasSprite,
    )>
) {
    for (mut timer, mut sprite) in query.iter_mut() {
        timer.0.tick(time.delta());
        
        if timer.0.just_finished() {
            sprite.index = (sprite.index + 1) % 8;  // Cycle through 8 frames
        }
    }
}
```

This cycles through the first 8 frames repeatedly, creating a looping animation.

## Dynamic Texture Atlases

Sometimes sprites come as individual files. Bevy can combine them into a texture atlas at runtime:

```rust
fn build_atlas(
    mut texture_atlases: ResMut<Assets<TextureAtlas>>,
    mut textures: ResMut<Assets<Image>>,
    asset_server: Res<AssetServer>
) {
    let mut atlas_builder = TextureAtlasBuilder::default();
    
    // Add individual sprite files
    let sprite1 = asset_server.load("sprite1.png");
    let sprite2 = asset_server.load("sprite2.png");
    let sprite3 = asset_server.load("sprite3.png");
    
    atlas_builder.add_texture(sprite1.clone(), textures.get(&sprite1).unwrap());
    atlas_builder.add_texture(sprite2.clone(), textures.get(&sprite2).unwrap());
    atlas_builder.add_texture(sprite3.clone(), textures.get(&sprite3).unwrap());
    
    let atlas = atlas_builder.finish(&mut textures).unwrap();
    let atlas_handle = texture_atlases.add(atlas);
}
```

This packs the sprites efficiently, reducing draw calls while keeping your asset files separate during development.

## Layer Ordering

2D games need control over what renders in front of what. Bevy uses the Z component of transforms for depth sorting:

```rust
// Background layer
commands.spawn(SpriteBundle {
    texture: asset_server.load("background.png"),
    transform: Transform::from_xyz(0.0, 0.0, 0.0),  // Z = 0
    ..Default::default()
});

// Player layer
commands.spawn(SpriteBundle {
    texture: asset_server.load("player.png"),
    transform: Transform::from_xyz(0.0, 0.0, 1.0),  // Z = 1, renders in front
    ..Default::default()
});

// UI layer
commands.spawn(SpriteBundle {
    texture: asset_server.load("health_bar.png"),
    transform: Transform::from_xyz(0.0, 0.0, 2.0),  // Z = 2, renders on top
    ..Default::default()
});
```

Higher Z values render in front. Use this to organize backgrounds, game objects, and foreground elements.

## Sprite Anchors

By default, sprites rotate and scale around their center. Change this with the anchor point:

```rust
sprite.anchor = Anchor::BottomLeft;  // Anchor at bottom-left corner
```

Available anchors include `Center`, `TopLeft`, `TopRight`, `BottomLeft`, `BottomRight`, and more. Custom anchors use `Anchor::Custom(Vec2)` with values from -0.5 to 0.5.

## Batch Rendering

Bevy automatically batches sprites that share the same texture and material. To maximize performance:

**Use sprite sheets** - Multiple sprites from one texture batch together.

**Share textures** - Reuse the same texture handles across entities.

**Minimize state changes** - Group sprites by texture when possible.

The renderer handles batching automatically, but these practices help it work efficiently.

## Common 2D Patterns

**Tiled backgrounds** - Spawn a grid of sprites for level backgrounds:

```rust
for x in 0..10 {
    for y in 0..10 {
        commands.spawn(SpriteBundle {
            texture: tile_texture.clone(),
            transform: Transform::from_xyz(x as f32 * 32.0, y as f32 * 32.0, 0.0),
            ..Default::default()
        });
    }
}
```

**Particle effects** - Spawn many small sprites with velocities:

```rust
for _ in 0..50 {
    commands.spawn((
        SpriteBundle {
            texture: particle_texture.clone(),
            sprite: Sprite {
                custom_size: Some(Vec2::splat(4.0)),
                ..Default::default()
            },
            transform: Transform::from_xyz(x, y, 0.0),
            ..Default::default()
        },
        Velocity(Vec2::new(
            rand::random::<f32>() * 100.0 - 50.0,
            rand::random::<f32>() * 100.0 - 50.0,
        )),
    ));
}
```

**Sprite trails** - Store previous positions and render fading sprites:

```rust
#[derive(Component)]
struct Trail {
    positions: Vec<Vec2>,
    max_length: usize,
}
```

## Pixel Perfect Rendering

For pixel art games, disable texture filtering and use integer positions:

```rust
// Set texture filtering in the Image
image.sampler_descriptor = ImageSampler::nearest();

// Use integer coordinates
transform.translation = Vec3::new(x.round(), y.round(), z);
```

This keeps pixels crisp without blurring.

## Performance Tips

**Minimize sprite count** - Fewer sprites mean fewer draw calls. Use sprite sheets and combine where possible.

**Cull off-screen sprites** - Bevy's frustum culling handles this automatically, but you can add custom culling for very large worlds.

**Limit alpha blending** - Transparent sprites require sorting and are more expensive than opaque ones.

**Reuse handles** - Clone asset handles instead of loading the same texture multiple times.

Bevy's 2D rendering handles sprites efficiently out of the box. Understanding sprite sheets, depth sorting, and batching will help you build performant 2D games.

