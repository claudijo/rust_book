# Scenes

Scenes are a way to compose pieces of your game/app ahead of time. In Bevy, Scenes are simply a collection of entities and components. A Scene can be "spawned" into a World any number of times. "Spawning" copies the Scene's entities and components into the given World.

Scenes can also be saved to and loaded from "scene files". One of the primary goals of the future "Bevy Editor" will be to make it easy to compose scene files visually.

## File Format

Scene files are saved and loaded as a flat list of entities and components:

```ron
[
  (
    entity: 328997855,
    components: [
      {
        "type": "Position",
        "map": { "x": 3.0, "y": 4.0 },
      },
    ],
  ),
  (
    entity: 404566393,
    components: [
      {
        "type": "Position",
        "map": { "x": 1.0, "y": 2.0 },
      },
      {
        "type": "Name",
        "map": { "value": "Carter" },
      },
    ],
  ),
]
```

The numbers assigned to the entity fields are the entity's id, which are completely optional. If no entity id is provided, one will be randomly generated when the scene is loaded. We have plans to improve this format in the future to be more ergonomic, indent entity hierarchies, and support nested scenes.

## Loading and Instancing

Scenes can be added to a World using the SceneSpawner resource. Spawning can be done with either `SceneSpawner::load` or `SceneSpawner::instance`.

- **"Loading"** a Scene preserves the entity IDs in it. This is useful for something like a save file where you want entity ids to be constant and changes to be applied on top of entities already in the world.
- **"Instancing"** adds entities to the World with brand-new IDs, which allows multiple "instances" of a scene to exist in the same World.

```rust
fn load_scene_system(
    asset_server: Res<AssetServer>,
    mut scene_spawner: ResMut<SceneSpawner>
) {
    // Scenes are loaded just like any other asset.
    let scene: Handle<Scene> = asset_server.load("my_scene.scn").unwrap();
    // Spawns the scene with entity ids preserved
    scene_spawner.load(scene);
    // Spawns the scene with new entity ids
    scene_spawner.instance(scene);
}
```

## Saving ECS Worlds To Scenes

Any ECS World can be converted to a scene:

```rust
let scene = Scene::from_world(&world, &component_type_registry);
```

You can then convert the scene to a RON formatted string:

```rust
let ron_string = scene.serialize_ron(&property_type_registry)?;
```

## Hot Scene Reloading

Changes to Scene files can be automatically applied to spawned Scenes at runtime. This allows for immediate feedback without restarts or recompiles.

> **Note**: Scene changes are effectively applied instantaneously.

## How This Works

Scenes are built on top of Bevy's Property and Asset systems. Components can be used in scenes provided they derive the Properties trait. Properties are what enable scene serialization, deserialization, and patching changes at runtime.

