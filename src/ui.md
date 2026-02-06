# User Interface

Every game needs UI: health bars, menus, inventory screens, dialog boxes. Bevy's UI system uses flexbox layout, integrates directly with the ECS, and supports both immediate and retained UI patterns.

## UI as Entities

In Bevy, UI elements are entities with specialized components. A button is an entity. A text label is an entity. A panel containing other elements is an entity. This means UI benefits from all of Bevy's ECS features: queries, change detection, events, hierarchies, and more.

The foundation of any UI element is a node - a rectangle positioned using flexbox layout:

```rust
fn spawn_ui(mut commands: Commands) {
    commands.spawn(NodeBundle {
        style: Style {
            width: Val::Px(200.0),
            height: Val::Px(100.0),
            ..Default::default()
        },
        background_color: Color::rgb(0.1, 0.1, 0.1).into(),
        ..Default::default()
    });
}
```

This creates a 200x100 pixel gray rectangle. Not exciting yet, but it's the building block for everything else.

## Flexbox Layout

Bevy uses flexbox for layout, the same model used in modern web development. If you know CSS flexbox, you already understand Bevy UI layout. If not, the basics are simple: containers arrange children either horizontally or vertically, with control over spacing, alignment, and sizing.

Center two boxes within a parent:

```rust
commands
    .spawn(NodeBundle {
        style: Style {
            width: Val::Percent(100.0),
            height: Val::Percent(100.0),
            justify_content: JustifyContent::Center,
            align_items: AlignItems::Center,
            ..Default::default()
        },
        background_color: Color::rgb(0.04, 0.04, 0.04).into(),
        ..Default::default()
    })
    .with_children(|parent| {
        parent.spawn(NodeBundle {
            style: Style {
                width: Val::Px(80.0),
                height: Val::Px(80.0),
                margin: UiRect::all(Val::Px(10.0)),
                ..Default::default()
            },
            background_color: Color::rgb(0.1, 0.1, 0.8).into(),
            ..Default::default()
        });
        
        parent.spawn(NodeBundle {
            style: Style {
                width: Val::Px(80.0),
                height: Val::Px(80.0),
                margin: UiRect::all(Val::Px(10.0)),
                ..Default::default()
            },
            background_color: Color::rgb(0.8, 0.1, 0.1).into(),
            ..Default::default()
        });
    });
```

The parent uses `justify_content` and `align_items` to center its children both horizontally and vertically. The children render as blue and red boxes side by side.

## Positioning Nodes

Flexbox handles relative positioning by default - children flow based on the container's rules. For precise control, use absolute positioning:

```rust
commands.spawn(NodeBundle {
    style: Style {
        width: Val::Px(200.0),
        height: Val::Px(100.0),
        position_type: PositionType::Absolute,
        left: Val::Px(10.0),
        top: Val::Px(10.0),
        ..Default::default()
    },
    background_color: Color::rgb(0.1, 0.1, 0.8).into(),
    ..Default::default()
});
```

Absolute positioning removes the node from flexbox flow and positions it relative to its parent's top-left corner. This is useful for floating elements like tooltips or draggable windows.

## Text

Text elements display strings with configurable fonts, sizes, and colors:

```rust
fn spawn_text(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn(TextBundle {
        text: Text::from_section(
            "Hello, Bevy!",
            TextStyle {
                font: asset_server.load("fonts/FiraSans-Bold.ttf"),
                font_size: 40.0,
                color: Color::WHITE,
            },
        ),
        ..Default::default()
    });
}
```

Text automatically wraps within its container and respects alignment settings.

## Rich Text

A single text entity can contain multiple sections with different styles:

```rust
commands.spawn(TextBundle {
    text: Text::from_sections([
        TextSection::new(
            "Health: ",
            TextStyle {
                font: asset_server.load("fonts/FiraSans-Bold.ttf"),
                font_size: 30.0,
                color: Color::WHITE,
            },
        ),
        TextSection::new(
            "100",
            TextStyle {
                font: asset_server.load("fonts/FiraMono-Medium.ttf"),
                font_size: 30.0,
                color: Color::GREEN,
            },
        ),
    ]),
    ..Default::default()
});
```

This renders "Health: " in one font and "100" in another with a different color. Update individual sections by querying the Text component and modifying specific sections.

## Images

Display textures in UI with ImageBundle:

```rust
fn spawn_image(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn(ImageBundle {
        style: Style {
            width: Val::Px(150.0),
            height: Val::Auto,
            ..Default::default()
        },
        image: asset_server.load("icon.png").into(),
        ..Default::default()
    });
}
```

Set width or height to `Auto` to maintain the texture's aspect ratio.

## Buttons

Buttons combine visual nodes with interaction tracking:

```rust
fn spawn_button(mut commands: Commands) {
    commands.spawn(ButtonBundle {
        style: Style {
            width: Val::Px(150.0),
            height: Val::Px(65.0),
            justify_content: JustifyContent::Center,
            align_items: AlignItems::Center,
            ..Default::default()
        },
        background_color: Color::rgb(0.15, 0.15, 0.15).into(),
        ..Default::default()
    })
    .with_children(|parent| {
        parent.spawn(TextBundle::from_section(
            "Click Me",
            TextStyle {
                font_size: 30.0,
                color: Color::WHITE,
                ..Default::default()
            },
        ));
    });
}
```

The button itself is a container, and the text is a child. This pattern separates the clickable area from the visual content.

## Button Interaction

Query the `Interaction` component to respond to button events:

```rust
fn button_system(
    mut interaction_query: Query<
        (&Interaction, &mut BackgroundColor),
        Changed<Interaction>
    >
) {
    for (interaction, mut color) in interaction_query.iter_mut() {
        match *interaction {
            Interaction::Pressed => {
                *color = Color::rgb(0.35, 0.35, 0.35).into();
                println!("Button pressed!");
            }
            Interaction::Hovered => {
                *color = Color::rgb(0.25, 0.25, 0.25).into();
            }
            Interaction::None => {
                *color = Color::rgb(0.15, 0.15, 0.15).into();
            }
        }
    }
}
```

The `Changed<Interaction>` filter ensures this only runs when interaction state changes, avoiding unnecessary work.

## Building a UI Layout

Let's build a practical game HUD:

```rust
fn spawn_hud(mut commands: Commands, asset_server: Res<AssetServer>) {
    // Root node - fills the screen
    commands
        .spawn(NodeBundle {
            style: Style {
                width: Val::Percent(100.0),
                height: Val::Percent(100.0),
                justify_content: JustifyContent::SpaceBetween,
                ..Default::default()
            },
            ..Default::default()
        })
        .with_children(|parent| {
            // Top-left corner - health display
            parent.spawn(NodeBundle {
                style: Style {
                    width: Val::Px(200.0),
                    height: Val::Px(50.0),
                    margin: UiRect::all(Val::Px(10.0)),
                    padding: UiRect::all(Val::Px(10.0)),
                    ..Default::default()
                },
                background_color: Color::rgba(0.0, 0.0, 0.0, 0.5).into(),
                ..Default::default()
            })
            .with_children(|parent| {
                parent.spawn(TextBundle::from_section(
                    "Health: 100",
                    TextStyle {
                        font: asset_server.load("fonts/FiraSans-Bold.ttf"),
                        font_size: 30.0,
                        color: Color::WHITE,
                    },
                ));
            });
            
            // Top-right corner - score display
            parent.spawn(NodeBundle {
                style: Style {
                    width: Val::Px(200.0),
                    height: Val::Px(50.0),
                    margin: UiRect::all(Val::Px(10.0)),
                    padding: UiRect::all(Val::Px(10.0)),
                    ..Default::default()
                },
                background_color: Color::rgba(0.0, 0.0, 0.0, 0.5).into(),
                ..Default::default()
            })
            .with_children(|parent| {
                parent.spawn(TextBundle::from_section(
                    "Score: 0",
                    TextStyle {
                        font: asset_server.load("fonts/FiraSans-Bold.ttf"),
                        font_size: 30.0,
                        color: Color::WHITE,
                    },
                ));
            });
        });
}
```

## Updating UI at Runtime

UI elements are entities, so update them like any other component:

```rust
#[derive(Component)]
struct HealthText;

fn update_health_display(
    player_health: Query<&Health, With<Player>>,
    mut text_query: Query<&mut Text, With<HealthText>>
) {
    if let Ok(health) = player_health.get_single() {
        for mut text in text_query.iter_mut() {
            text.sections[0].value = format!("Health: {}", health.current);
        }
    }
}
```

Tag UI entities with marker components to query them specifically.

## Clipping Content

The `Overflow::Hidden` property clips content that extends beyond a node's bounds:

```rust
commands.spawn(NodeBundle {
    style: Style {
        width: Val::Px(300.0),
        height: Val::Px(200.0),
        overflow: Overflow::clip(),
        ..Default::default()
    },
    background_color: Color::rgb(0.1, 0.1, 0.1).into(),
    ..Default::default()
})
.with_children(|parent| {
    // This content will be clipped if too large
    parent.spawn(TextBundle::from_section(
        "Very long text that might overflow the container...",
        TextStyle { /* ... */ },
    ));
});
```

This is essential for scrollable lists, text boxes, and chat windows.

## Text in World Space

While UI text appears in screen space, you can also render text in the game world using Text2dBundle:

```rust
commands.spawn(Text2dBundle {
    text: Text::from_section(
        "3D Label",
        TextStyle {
            font: asset_server.load("fonts/FiraSans-Bold.ttf"),
            font_size: 60.0,
            color: Color::WHITE,
        },
    ),
    transform: Transform::from_xyz(0.0, 2.0, 0.0)
        .with_rotation(Quat::from_rotation_y(std::f32::consts::PI / 4.0)),
    ..Default::default()
});
```

This text exists in 3D space, rotates with the camera, and can be transformed like any other entity. It's perfect for labels, damage numbers, or floating UI elements.

## Z-Index and Layering

UI elements render in a specific order. By default, children render on top of parents, and later siblings render on top of earlier ones. This hierarchy-based ordering works for most UIs, but some scenarios need explicit control.

### Z-Index Control

The `ZIndex` component provides two modes for controlling rendering order:

**Local Z-Index** - Orders elements relative to siblings:

```rust
commands.spawn((
    NodeBundle {
        background_color: Color::BLUE.into(),
        ..Default::default()
    },
    ZIndex::Local(-1),  // Render behind siblings
));
```

**Global Z-Index** - Orders elements relative to the entire UI root:

```rust
commands.spawn((
    NodeBundle {
        background_color: Color::RED.into(),
        ..Default::default()
    },
    ZIndex::Global(100),  // Render on top of everything
));
```

Global z-index lets elements "escape" their parent's ordering and render relative to the entire UI. This is useful for tooltips, modal dialogs, or overlays that need to appear above all other UI.

### Ordering Rules

1. Elements with higher z-index values render on top
2. Within the same z-level, hierarchy order determines rendering (later children on top)
3. Global z-index overrides local hierarchy completely

**Example:** If you have nested UI elements where a child needs to render behind its parent, use a negative local z-index. If you need a notification to appear above everything regardless of hierarchy, use a high global z-index.

## Responsive UI

Size nodes with percentages to scale with screen resolution:

```rust
commands.spawn(NodeBundle {
    style: Style {
        width: Val::Percent(80.0),  // 80% of parent width
        height: Val::Px(100.0),     // Fixed 100 pixels
        ..Default::default()
    },
    ..Default::default()
});
```

Combine fixed and percentage sizing to create UIs that adapt to different screen sizes.

### Global UI Scale

Configure a global scale multiplier for all UI elements using `UiScale`:

```rust
fn main() {
    App::new()
        .insert_resource(UiScale { scale: 2.0 })  // 2x larger UI
        .add_plugins(DefaultPlugins)
        .run();
}
```

This scales all UI dimensions uniformly, useful for:
- Accessibility options (larger UI for visibility)
- High-DPI display adaptation
- User preference settings
- Platform-specific scaling (mobile vs desktop)

The scale applies to all pixel values in UI, making it easy to provide user-configurable UI sizing without redesigning layouts.

## Design Patterns

UI built with the ECS encourages certain patterns:

**Marker components** - Tag UI elements for specific queries:
```rust
#[derive(Component)]
struct HealthBar;

#[derive(Component)]
struct MenuButton;
```

**Bundle reuse** - Create functions that return configured bundles:
```rust
fn styled_button(text: &str) -> ButtonBundle {
    ButtonBundle {
        style: Style {
            width: Val::Px(150.0),
            height: Val::Px(65.0),
            justify_content: JustifyContent::Center,
            align_items: AlignItems::Center,
            ..Default::default()
        },
        background_color: Color::rgb(0.15, 0.15, 0.15).into(),
        ..Default::default()
    }
}
```

**State-driven UI** - Spawn and despawn UI based on game state:
```rust
fn setup_menu(mut commands: Commands) {
    commands
        .spawn(NodeBundle { /* ... */ })
        .insert(MenuRoot);  // Tag for cleanup
}

fn cleanup_menu(mut commands: Commands, query: Query<Entity, With<MenuRoot>>) {
    for entity in query.iter() {
        commands.entity(entity).despawn_recursive();
    }
}
```

## Performance Considerations

UI rendering is efficient, but keep a few things in mind:

**Minimize changes** - Only update text and colors when they actually change. Use change detection to avoid unnecessary updates.

**Batch similar elements** - Group elements with the same materials to reduce draw calls.

**Don't over-nest** - Deep hierarchies work but aren't free. Keep nesting reasonable.

**Reuse fonts** - Load fonts once and share handles rather than loading the same font multiple times.

## Best Practices

**Use marker components** - Tag UI elements for easy querying and modification.

**Separate data from presentation** - Store game state separately from UI. Update UI based on state changes.

**Design for multiple resolutions** - Use percentage sizing and test on different screen sizes.

**Provide visual feedback** - Show hover and press states for interactive elements.

**Consider accessibility** - Use reasonable font sizes and color contrast.

Bevy's UI system integrates naturally with the rest of the engine. Understanding flexbox layout and ECS patterns will let you build complex, interactive interfaces efficiently.

