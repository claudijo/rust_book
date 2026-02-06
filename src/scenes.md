# Scenes

Scenes organize game content into reusable, composable pieces. A scene is a collection of entities and their components that can be spawned into your world multiple times.

## What Scenes Solve

Building complex game worlds entity-by-entity in code becomes tedious. You want to define level layouts, character configurations, and UI structures once and reuse them. Scenes provide this abstraction.

Think of a scene as a template. Define enemies with their components, behaviors, and appearance once. Spawn that enemy scene whenever you need it. Create level sections as scenes and compose larger levels from these pieces.

## Scene Structure

Scenes store entities and components in a serializable format. Load scenes from files or build them programmatically:

```rust
fn create_enemy_scene(world: &mut World) -> Scene {
    let mut builder = world.resource::<SceneBuilder>();
    
    builder
        .spawn((
            Enemy,
            Health { current: 100.0, max: 100.0 },
            Transform::from_xyz(0.0, 0.0, 0.0),
        ))
        .with_children(|parent| {
            parent.spawn((
                SpriteBundle {
                    texture: /* ... */,
                    ..Default::default()
                },
            ));
        });
    
    builder.build()
}
```

## Loading Scenes

Load scenes from files like any asset:

```rust
fn load_level(mut commands: Commands, asset_server: Res<AssetServer>) {
    let scene = asset_server.load("levels/level_1.scn.ron");
    commands.spawn(SceneBundle { scene, ..Default::default() });
}
```

Bevy spawns the scene's entities into your world automatically when the asset finishes loading.

## Scene Files

Scenes serialize to RON (Rusty Object Notation) format, a human-readable data format. The scene format is clean and concise:

```ron
(
  entities: {
    0: (
      components: {
        "game::Player": (
          name: "Alice",
          position: (
            x: 0.0,
            y: 0.0,
          ),
        ),
        "game::Health": (
          current: 100,
          max: 100,
        ),
        "game::Team": A,
      },
    ),
    1: (
      components: {
        "game::Player": (
          name: "Bob",
          position: (
            x: 10.0,
            y: 0.0,
          ),
        ),
        "game::Health": (
          current: 80,
          max: 100,
        ),
        "game::Team": B,
      },
    ),
  },
)
```

The format is intuitive: entities are stored in a map by ID, and components are listed with their full type paths. Struct fields appear as clean key-value pairs, tuples are parenthesized lists, and enums show their variant names.

### Binary Scene Format

For production builds, use binary scene files (`.scn` extension) for faster loading and smaller file sizes:

```rust
// Save as binary
let scene = DynamicScene { /* ... */ };
let bytes = scene.serialize_binary(&type_registry).unwrap();
std::fs::write("level.scn", bytes).unwrap();

// Load binary scenes like normal
let scene = asset_server.load("level.scn");
```

Binary scenes load significantly faster than text scenes, making them ideal for shipped games. Use text scenes during development for easy editing, then convert to binary for release builds.

## Spawning Scenes

Spawn scenes as entities:

```rust
fn spawn_enemies(mut commands: Commands, enemy_scene: Res<EnemyScene>) {
    // Spawn the scene
    commands.spawn(SceneBundle {
        scene: enemy_scene.handle.clone(),
        transform: Transform::from_xyz(100.0, 0.0, 0.0),
        ..Default::default()
    });
    
    // Spawn it again elsewhere
    commands.spawn(SceneBundle {
        scene: enemy_scene.handle.clone(),
        transform: Transform::from_xyz(-100.0, 0.0, 0.0),
        ..Default::default()
    });
}
```

Each spawn creates independent entities. Modifying one instance doesn't affect others.

## Creating Scenes from Worlds

Extract parts of your world into a scene:

```rust
fn save_level(world: &World, type_registry: &TypeRegistry) {
    // Create scene from world
    let scene = DynamicScene::from_world(world, type_registry);
    
    // Serialize to RON
    let serialized = scene.serialize_ron(type_registry).unwrap();
    
    // Write to file
    std::fs::write("level.scn.ron", serialized).unwrap();
}
```

## Building Scenes Dynamically

Create scenes from your running game world using `DynamicSceneBuilder`:

```rust
fn save_game_state(world: &World) -> DynamicScene {
    DynamicSceneBuilder::from_world(world)
        .extract_entities(world.iter_entities().map(|e| e.id()))
        .build()
}
```

This captures entities and their components, creating a reusable scene. The builder provides fine-grained control over what gets saved.

## Selective Scene Extraction

Extract only specific entities using queries:

```rust
fn save_player_state(
    world: &World,
    player_query: Query<Entity, With<Player>>
) -> DynamicScene {
    // Build scene with only player entities
    DynamicSceneBuilder::from_world(world)
        .extract_entities(player_query.iter())
        .build()
}
```

Or extract individual entities:

```rust
let mut builder = DynamicSceneBuilder::from_world(world);
builder.extract_entity(player_entity);
builder.extract_entity(companion_entity);
let scene = builder.build();
```

This focused extraction is perfect for save systems, prefab creation, or exporting specific game objects. Extract only what you need, keeping scenes small and fast to load.

## Hot Reloading

Scene files support hot reloading during development. Edit a scene file and see changes immediately:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_startup_system(load_scene)
        .run();
}

fn load_scene(mut commands: Commands, asset_server: Res<AssetServer>) {
    // Enable asset watching
    asset_server.watch_for_changes().unwrap();
    
    let scene = asset_server.load("test_scene.scn.ron");
    commands.spawn(SceneBundle { scene, ..Default::default() });
}
```

Edit `test_scene.scn.ron` while your game runs. The scene updates automatically without restarting.

## Component Requirements

Components must be serializable to appear in scenes. Derive the necessary traits:

```rust
#[derive(Component, Reflect, Default)]
#[reflect(Component)]
struct Health {
    current: f32,
    max: f32,
}
```

Register the component with the type registry:

```rust
app.register_type::<Health>();
```

Now `Health` can be saved to and loaded from scene files.

## Scene Hierarchies

Scenes preserve parent-child relationships:

```rust
fn create_character_scene() {
    commands
        .spawn((
            Character,
            Transform::default(),
        ))
        .with_children(|parent| {
            parent.spawn(SpriteBundle {
                texture: character_sprite,
                ..Default::default()
            });
            
            parent.spawn(SpriteBundle {
                texture: weapon_sprite,
                transform: Transform::from_xyz(1.0, 0.0, 0.0),
                ..Default::default()
            });
        });
}
```

When saved to a scene, the hierarchy remains intact. Loading the scene recreates the complete structure.

## Use Cases

**Level composition** - Build levels from reusable room sections. Each room is a scene. Combine them to create complete levels.

**Enemy templates** - Define enemy types once. Spawn them throughout your game with consistent behavior and appearance.

**Save systems** - Serialize player state and world data. Load it later to resume gameplay.

**Prefabs** - Store complex entity configurations. Spawn them programmatically without rebuilding in code.

**UI layouts** - Define menu structures in scene files. Load different menus based on game state.

## Best Practices

**Keep scenes focused** - Small, specific scenes are easier to manage than large, monolithic ones.

**Use composition** - Build complex scenes from simpler ones rather than creating everything at once.

**Version scene files** - Track scene file changes in version control to collaborate on level design.

**Test scene loading** - Ensure scenes work across different game states and loading orders.

**Handle missing components** - Not all components serialize. Design systems to handle partially loaded scenes gracefully.

Scenes provide a powerful way to organize game content. They enable iteration without recompilation, support team collaboration through file formats, and make complex content manageable through composition.

