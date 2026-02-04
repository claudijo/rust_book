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

## Bevy 0.2 Improvements

**Changes in Bevy 0.2:**

### Multiline Text Support

The `DrawableText` component now supports multiline text rendering.

### Default Node Size

The default node size changed from `Undefined` to `Auto` to match the Stretch implementation. This provides more intuitive default behavior.

### Component Bundle Cloning

UI component bundles now derive `Clone`, making it easier to reuse UI configurations:

```rust
let button_bundle = ButtonComponents {
    // ... configuration
};

// Can now clone the bundle
commands.spawn(button_bundle.clone());
```

### Font Atlas Improvements

Fixed font atlas overflow issues that could occur with certain font sizes and character sets.

## Bevy 0.6 Improvements

**Added in Bevy 0.6:**

### Overflow::Hidden

UI nodes now respect the flexbox `Overflow::Hidden` property, which cuts off child content that extends beyond the parent's bounds:

```rust
commands.spawn_bundle(NodeBundle {
    style: Style {
        size: Size::new(Val::Px(200.0), Val::Px(100.0)),
        overflow: Overflow::Hidden,  // Cut off children outside bounds
        flex_direction: FlexDirection::Column,
        ..Default::default()
    },
    color: Color::rgb(0.1, 0.1, 0.1).into(),
    ..Default::default()
})
.with_children(|parent| {
    // This content will be clipped if it exceeds parent size
    parent.spawn_bundle(TextBundle {
        text: Text::with_section(
            "This is a very long text that might overflow...",
            TextStyle {
                font: asset_server.load("fonts/FiraSans-Bold.ttf"),
                font_size: 20.0,
                color: Color::WHITE,
            },
            Default::default(),
        ),
        ..Default::default()
    });
});
```

**Use cases:**
- Scrollable lists
- Text boxes with size limits
- Clipped image viewers
- Chat windows

### Text2D Transforms

Text2d now supports arbitrary transformations using the Transform component:

```rust
commands.spawn_bundle(Text2dBundle {
    text: Text::with_section(
        "Rotated Text!",
        TextStyle {
            font: asset_server.load("fonts/FiraSans-Bold.ttf"),
            font_size: 60.0,
            color: Color::WHITE,
        },
        TextAlignment {
            horizontal: HorizontalAlign::Center,
            ..Default::default()
        },
    ),
    transform: Transform {
        translation: Vec3::new(0.0, 0.0, 1.0),
        rotation: Quat::from_rotation_z(std::f32::consts::PI / 4.0),  // 45 degree rotation
        scale: Vec3::splat(1.5),
    },
    ..Default::default()
});
```

**Capabilities:**
- Rotate text at any angle
- Scale text (though font_size is usually better for crisp rendering)
- Position text in 2D world space
- Apply any transform (even 3D transformations!)

**Note:** While `Transform::scale` works, adjusting `font_size` directly is generally better for crisp text rendering.

## Bevy 0.5 Text Improvements

**Added in Bevy 0.5:**

### Rich Text

Text can now have multiple "sections", each with their own style and formatting:

```rust
commands.spawn_bundle(TextBundle {
    text: Text {
        sections: vec![
            TextSection {
                value: "Score: ".to_string(),
                style: TextStyle {
                    font: asset_server.load("fonts/FiraSans-Bold.ttf"),
                    font_size: 60.0,
                    color: Color::WHITE,
                },
            },
            TextSection {
                value: "0".to_string(),
                style: TextStyle {
                    font: asset_server.load("fonts/FiraMono-Medium.ttf"),
                    font_size: 60.0,
                    color: Color::GOLD,
                },
            },
            TextSection {
                value: " points".to_string(),
                style: TextStyle {
                    font: asset_server.load("fonts/FiraSans-Bold.ttf"),
                    font_size: 40.0,
                    color: Color::GRAY,
                },
            },
        ],
        ..Default::default()
    },
    ..Default::default()
});
```

**Benefits:**
- Different fonts per section
- Different sizes per section  
- Different colors per section
- All within a single Text component
- Respects text layout rules

**Use cases:**
- Colored/highlighted text
- Mixed font UI (labels + values)
- Formatted game text
- Dynamic text with varying styles

### HIDPI Text

**Added in Bevy 0.5**

Text is now automatically rendered according to the monitor's scale factor, providing crisp text at any resolution:

- ✅ Automatically detects DPI scaling
- ✅ Renders at native resolution
- ✅ Crisp on retina/HIDPI displays
- ✅ No configuration needed

Text now looks sharp on both standard and high-DPI displays!

