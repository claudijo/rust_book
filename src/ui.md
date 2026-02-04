# Bevy UI

Bevy has a custom UI system based on the "flex box" model.

## Design Philosophy

Bevy UI was built to fully embrace the Bevy way of doing things. While there are many great pre-made UI solutions in the Rust ecosystem, integrating them would compromise either the UI framework's design or Bevy's ECS approach.

Bevy UI directly uses the existing ECS, hierarchy, transform, event, asset, and scene systems at the core of Bevy. Because of this, Bevy UI automatically gets features like:

- Hot-reloading of UI scene files
- Async texture loading
- Change detection

A shared architecture means improvements to any of these systems feed directly into Bevy UI.

> **Note**: We are in the experimental stages and expect some things to change. Currently the best way to compose Bevy UIs is with code, but we are designing a new file format for scenes that will make declarative, file-based UI composition much nicer.

## Building Blocks

In Bevy, a UI element is just an ECS Entity with a Node component. Nodes are rectangles with a width and height, and are positioned using the same Transform component used elsewhere in Bevy. The Style component is used to determine how the Node is rendered, sized, and positioned.

The easiest way to add a new node:

```rust
commands.spawn(NodeComponents::default())
```

NodeComponents is a "component bundle", which Bevy uses to make spawning entities of various "types" easier.

## Layout

For layout, Bevy uses Stretch - a 100% Rust flexbox implementation. Stretch provides the algorithms for positioning rectangles in 2D space according to the flexbox spec. Bevy exposes flex properties inside the Style component and renders rectangles with the positions and sizes that Stretch outputs. Bevy uses its own z-layering algorithm to "stack" elements on top of each other (basically the same one that HTML/CSS uses).

### Relative Positioning

Nodes are positioned relative to each other by default:

```rust
commands
    .spawn(NodeComponents {
        style: Style {
            size: Size::new(Val::Px(100.0), Val::Px(100.0)),
            ..Default::default()
        },
        material: materials.add(Color::rgb(0.08, 0.08, 1.0).into()),
        ..Default::default()
    })
    .spawn(NodeComponents {
        style: Style {
            size: Size::new(Val::Percent(40.0), Val::Percent(40.0)),
            ..Default::default()
        },
        material: materials.add(Color::rgb(1.0, 0.08, 0.08).into()),
        ..Default::default()
    });
```

### Absolute Positioning

You can "absolutely" position a Node relative to its parent's corners:

```rust
commands.spawn(NodeComponents {
    style: Style {
        size: Size::new(Val::Percent(40.0), Val::Percent(40.0)),
        position_type: PositionType::Absolute,
        position: Rect {
            top: Val::Px(10.0),
            right: Val::Px(10.0),
            ..Default::default()
        },
        ..Default::default()
    },
    material: materials.add(Color::rgb(0.08, 0.08, 1.0).into()),
    ..Default::default()
});
```

### Parenting

Just like any other Entity, Nodes can have children. Children are positioned and scaled relative to their parent. By default, children will always appear in front of their parents.

```rust
commands
    .spawn(NodeComponents {
        style: Style {
            size: Size::new(Val::Percent(60.0), Val::Percent(60.0)),
            position_type: PositionType::Absolute,
            position: Rect {
                top: Val::Px(10.0),
                right: Val::Px(10.0),
                ..Default::default()
            },
            ..Default::default()
        },
        material: materials.add(Color::rgb(0.08, 0.08, 1.0).into()),
        ..Default::default()
    })
    .with_children(|parent| {
        parent.spawn(NodeComponents {
            style: Style {
                size: Size::new(Val::Px(50.0), Val::Px(50.0)),
                ..Default::default()
            },
            material: materials.add(Color::rgb(0.5, 0.5, 1.0).into()),
            ..Default::default()
        });
    });
```

## Flexbox

You can use all of the same "flex" properties you would use in a web context. Here is an example of how you would center two Nodes vertically and horizontally within their parent:

```rust
commands
    .spawn(NodeComponents {
        style: Style {
            size: Size::new(Val::Percent(100.0), Val::Percent(100.0)),
            justify_content: JustifyContent::Center,
            align_items: AlignItems::Center,
            ..Default::default()
        },
        material: materials.add(Color::rgb(0.04, 0.04, 0.04).into()),
        ..Default::default()
    })
    .with_children(|parent| {
        parent
            .spawn(NodeComponents {
                style: Style {
                    size: Size::new(Val::Px(80.0), Val::Px(80.0)),
                    ..Default::default()
                },
                material: materials.add(Color::rgb(0.08, 0.08, 1.0).into()),
                ..Default::default()
            })
            .spawn(NodeComponents {
                style: Style {
                    size: Size::new(Val::Px(80.0), Val::Px(80.0)),
                    ..Default::default()
                },
                material: materials.add(Color::rgb(1.0, 0.08, 0.08).into()),
                ..Default::default()
            });
    });
```

## Text and Images

Nodes can also have Text and Image components, which affect the inferred sizes of nodes.

### Text

```rust
commands.spawn(TextComponents {
    text: Text {
        value: "Hello from Bevy UI!".to_string(),
        font: asset_server.load("FiraSans-Bold.ttf").unwrap(),
        style: TextStyle {
            font_size: 25.0,
            color: Color::WHITE,
        },
    },
    ..Default::default()
});
```

### Images

```rust
commands.spawn(ImageComponents {
    style: Style {
        size: Size::new(Val::Px(200.0), Val::Auto),
        position_type: PositionType::Absolute,
        position: Rect {
            top: Val::Px(10.0),
            right: Val::Px(10.0),
            ..Default::default()
        },
        ..Default::default()
    },
    material: materials.add(asset_server.load("bevy_logo.png").unwrap().into()),
    ..Default::default()
});
```

## Interaction Events

Nodes with the Interaction component will track interaction state. You can easily build widgets like buttons this way.

Here is a system that only runs on Buttons where the Interaction state has changed:

```rust
fn system(_button: &Button, interaction: Mutated<Interaction>) {
    match *interaction {
        Interaction::Clicked => println!("clicked"),
        Interaction::Hovered => println!("hovered"),
        Interaction::None => {},
    }
}
```

