# Bevy Apps

A Bevy App is the entry point for any Bevy program. It manages your game's execution loop, plugin system, and core engine functionality.

## Creating an App

The simplest Bevy app does nothing but immediately terminate:

```rust
use bevy::prelude::*;

fn main() {
    App::new().run();
}
```

## Cross-Platform Support

Mobile platforms (Android and iOS) require platform-specific initialization code. Rather than writing this boilerplate yourself, use the `#[bevy_main]` attribute:

```rust
use bevy::prelude::*;

#[bevy_main]
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .run();
}
```

This works identically on all platforms: Windows, macOS, Linux, Android, iOS, and Web. The macro automatically generates the necessary platform-specific code at compile time.

## Default Plugins

The `DefaultPlugins` group includes everything you need for a complete game:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .run();
}
```

This adds the 2D/3D renderer, asset loading, UI system, windowing, input handling, audio, and more.

## Plugin Groups

Plugin groups organize related plugins and allow selective configuration. You can disable individual plugins from a group:

```rust
fn main() {
    App::new()
        .add_plugins_with(DefaultPlugins, |group| {
            group
                .disable::<RenderPlugin>()  // Headless server
                .disable::<AudioPlugin>()
        })
        .run();
}
```

### Plugin Settings

Configure plugins using the `.set()` method:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(WindowPlugin {
            window: WindowDescriptor {
                width: 1920.0,
                height: 1080.0,
                title: "My Game".to_string(),
                ..Default::default()
            },
            ..Default::default()
        }))
        .run();
}
```

Plugin settings are specified during initialization and can't be changed after the plugin is added. This distinguishes "initialization settings" from "runtime resources" that can be modified during gameplay.

Chain multiple `.set()` calls to configure different plugins:

```rust
App::new()
    .add_plugins(
        DefaultPlugins
            .set(WindowPlugin { /* ... */ })
            .set(AssetPlugin {
                watch_for_changes: true,
                ..Default::default()
            })
    )
    .run();
```

This pattern prevents the confusion of modifying settings that only apply during plugin initialization.
### Custom Plugin Groups

Create your own plugin groups to organize game-specific functionality:

```rust
pub struct GameplayPlugins;

impl PluginGroup for GameplayPlugins {
    fn build(&mut self, group: &mut PluginGroupBuilder) {
        group
            .add(PhysicsPlugin)
            .add(AIPlugin)
            .add(NetworkingPlugin)
            .add(SaveSystemPlugin);
    }
}

// Use your custom plugin group
app.add_plugins(MyGamePlugins);
```

This provides much more flexibility than the old static `add_default_plugins()` method.

## Manual Plugin Registration

You can also register the default Plugins manually:

```rust
fn main() {
    App::build()
        .add_plugin(CorePlugin::default())
        .add_plugin(InputPlugin::default())
        .add_plugin(WindowPlugin::default())
        .add_plugin(RenderPlugin::default())
        .add_plugin(UiPlugin::default())
        /* more plugins here ... omitted for brevity */
        .run();
}
```

Use your custom plugin group:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(GameplayPlugins)
        .run();
}
```

## Creating Plugins

Plugins encapsulate functionality that can be shared across projects. A plugin implements the `Plugin` trait:

```rust
pub struct PhysicsPlugin;

impl Plugin for PhysicsPlugin {
    fn build(&self, app: &mut App) {
        app
            .add_system(apply_gravity)
            .add_system(detect_collisions)
            .add_system(resolve_collisions);
    }
}
```

Add your plugin to the app:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugin(PhysicsPlugin)
        .run();
}
```

### Why Plugins?

Plugins provide several benefits:

**Organization** - Group related systems and resources together
**Reusability** - Share functionality across multiple projects
**Modularity** - Enable/disable features easily
**Dependencies** - Plugins can depend on other plugins

### Plugin Configuration

Plugins can accept configuration:

```rust
pub struct PhysicsPlugin {
    pub gravity: f32,
}

impl Plugin for PhysicsPlugin {
    fn build(&self, app: &mut App) {
        app
            .insert_resource(Gravity(self.gravity))
            .add_system(apply_gravity);
    }
}

// Use with custom configuration
app.add_plugin(PhysicsPlugin { gravity: -9.8 });
```
```

All engine and game logic is built using plugins. You can create your own plugins to organize your code.

## Modularity

Bevy's philosophy: **use only what you need, replace what you don't like**. You are free to add/remove plugins based on your project's unique needs. Most people will stick to `add_default_plugins()` for simplicity, at least initially.

## Bevy 0.2 Improvements

### Startup Stage Control

**Added in Bevy 0.2**

AppBuilder now has `add_startup_stage_before()` and `add_startup_stage_after()` methods for better control over startup system execution order:

```rust
fn main() {
    App::build()
        .add_default_plugins()
        .add_startup_stage_before("startup", "pre_startup")
        .add_startup_system_to_stage("pre_startup", early_setup.system())
        .run();
}
```

### System Initialization

**Fixed in Bevy 0.2**

App.schedule systems are now properly initialized when running the app, ensuring all systems are ready before the first frame.

