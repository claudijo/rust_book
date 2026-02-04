# Bevy Apps

A Bevy App is the starting point for any Bevy program.

## The Simplest App

**Bevy 0.6+:**
```rust
use bevy::prelude::*;

fn main() {
    App::new().run();
}
```

**Before 0.6:**
```rust
fn main() {
    App::build().run();
}
```

**Changed in Bevy 0.6:** `App::build()` → `App::new()`, and AppBuilder was merged into App.

This App pulls in no features and literally does nothing. Running the program would just immediately terminate.

## Cross-Platform Main Function

**Added in Bevy 0.4**

Most platforms (Windows, macOS, Linux, Web) work with normal main functions. However, some platforms (Android and iOS) require additional boilerplate. The `#[bevy_main]` attribute macro handles this automatically:

```rust
use bevy::prelude::*;

#[bevy_main]
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .run();
}
```

This single code works on **all platforms**: Windows, macOS, Linux, Android, iOS, and Web!

**Without `#[bevy_main]`:** You'd need platform-specific boilerplate for mobile platforms.

**With `#[bevy_main]`:** The macro inserts the necessary boilerplate automatically, enabling true "write once, run anywhere" for Bevy apps.

## Adding Default Plugins

**Bevy 0.5-0.6:**
```rust
fn main() {
    App::new()  // Changed from App::build() in 0.6
        .add_plugins(DefaultPlugins)
        .run();
}
```

**Bevy 0.3-0.4:**
```rust
fn main() {
    App::build()
        .add_plugins(DefaultPlugins)
        .run();
}
```

`DefaultPlugins` is a PluginGroup that adds all core features: 2D/3D renderer, asset loading, UI system, windows, input, etc.

## Plugin Groups

**Added in Bevy 0.3**

PluginGroups are ordered collections of plugins that can be individually enabled or disabled:

```rust
// Use all default plugins
app.add_plugins(DefaultPlugins);

// Disable specific plugins in a group
app.add_plugins_with(DefaultPlugins, |group| {
    group.disable::<RenderPlugin>()
         .disable::<AudioPlugin>()
});
```

### Creating Custom Plugin Groups

You can create your own plugin groups:

```rust
pub struct MyGamePlugins;

impl PluginGroup for MyGamePlugins {
    fn build(&mut self, group: &mut PluginGroupBuilder) {
        group.add(PhysicsPlugin)
             .add(NetworkingPlugin)
             .add(GameLogicPlugin);
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

## Creating Custom Plugins

**Bevy 0.6+:**
```rust
impl Plugin for MyGamePlugins {
    fn build(&self, app: &mut App) {  // App instead of AppBuilder
        app.add_system(my_system);
    }
}
```

**Before 0.6:**
```rust
impl Plugin for MyGamePlugins {
    fn build(&self, app: &mut AppBuilder) {
        app.add_system(my_system.system());
    }
}
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

