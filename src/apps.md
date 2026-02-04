# Bevy Apps

A Bevy App is the starting point for any Bevy program.

## The Simplest App

```rust
use bevy::prelude::*;

fn main() {
    App::build().run();
}
```

This App pulls in no features and literally does nothing. Running the program would just immediately terminate.

## Adding Default Plugins

```rust
fn main() {
    App::build()
        .add_default_plugins()
        .run();
}
```

`add_default_plugins()` adds all of the features you probably expect from a game engine: a 2D / 3D renderer, asset loading, a UI system, windows, input, etc.

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

All engine and game logic is built using plugins. You can create your own plugins to organize your code.

## Modularity

Bevy's philosophy: **use only what you need, replace what you don't like**. You are free to add/remove plugins based on your project's unique needs. Most people will stick to `add_default_plugins()` for simplicity, at least initially.

