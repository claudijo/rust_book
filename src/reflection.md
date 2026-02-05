# Reflection

Bevy's reflection system lets you inspect and modify Rust types at runtime. This enables dynamic behavior that wouldn't be possible with compile-time type information alone.

## Runtime Type Information

Rust is statically typed - the compiler needs to know types at compile time. But sometimes you need to work with types dynamically: loading data from files, building editor tools, or implementing plugin systems. Reflection bridges this gap.

With reflection, you can:
- Access struct fields by name at runtime
- Iterate over all fields in a struct
- Serialize and deserialize types automatically
- Call trait methods on unknown types
- Clone values without knowing their concrete type

## Basic Reflection

Derive `Reflect` to enable reflection on your types:

```rust
use bevy::prelude::*;

#[derive(Reflect)]
struct Player {
    name: String,
    health: f32,
    level: u32,
}

fn example() {
    let mut player = Player {
        name: "Alice".to_string(),
        health: 100.0,
        level: 5,
    };
    
    // Access fields by name
    let health = player.get_field::<f32>("health").unwrap();
    println!("Health: {}", health);
    
    // Modify fields by name
    *player.get_field_mut::<f32>("health").unwrap() = 75.0;
}
```

This works at runtime with string field names - something normal Rust can't do.

## Field Iteration

Inspect all fields dynamically:

```rust
fn inspect_fields(reflect_value: &dyn Reflect) {
    if let ReflectRef::Struct(struct_info) = reflect_value.reflect_ref() {
        for i in 0..struct_info.field_len() {
            let field_name = struct_info.name_at(i).unwrap();
            let field_value = struct_info.field_at(i).unwrap();
            
            println!("Field '{}' has value: {:?}", field_name, field_value);
        }
    }
}
```

This powers inspector UIs that display any struct's contents without custom code per type.

## Nested Field Access

Access deeply nested fields with path strings:

```rust
#[derive(Reflect)]
struct Inventory {
    items: Vec<Item>,
}

#[derive(Reflect)]
struct Item {
    name: String,
    quantity: u32,
}

fn example(inventory: &mut Inventory) {
    // Access nested field with path syntax
    let quantity = inventory
        .get_path::<u32>("items[0].quantity")
        .unwrap();
    
    println!("First item quantity: {}", quantity);
    
    // Modify nested field
    *inventory
        .get_path_mut::<u32>("items[0].quantity")
        .unwrap() = 5;
}
```

Path strings navigate through structs, tuples, and collections using intuitive syntax.

## Dynamic Values

Create and modify values without knowing their concrete types:

```rust
fn modify_player(player: &mut dyn Reflect) {
    // Create a dynamic struct with fields to update
    let mut dynamic_update = DynamicStruct::default();
    dynamic_update.insert("health", 50.0f32);
    dynamic_update.insert("level", 10u32);
    
    // Apply changes to the player
    player.apply(&dynamic_update);
}
```

This patches values onto existing types, updating only the fields present in the dynamic value.

## Serialization

Reflection enables generic serialization without manual Serde implementations:

```rust
fn save_player(player: &Player, registry: &TypeRegistry) -> String {
    let serializer = ReflectSerializer::new(player, registry);
    ron::ser::to_string_pretty(&serializer, Default::default()).unwrap()
}

fn load_player(data: &str, registry: &TypeRegistry) -> Player {
    let mut deserializer = ron::de::Deserializer::from_str(data).unwrap();
    let reflect_deserializer = ReflectDeserializer::new(registry);
    
    let reflected = reflect_deserializer.deserialize(&mut deserializer).unwrap();
    Player::from_reflect(reflected.as_reflect()).unwrap()
}
```

The type registry tracks which types are available, enabling safe deserialization.

## Type Registration

Register types to make them available for reflection:

```rust
fn setup(app: &mut App) {
    app.register_type::<Player>()
       .register_type::<Inventory>()
       .register_type::<Item>();
}
```

Registration adds types to the type registry, enabling serialization and other reflection features.

## Trait Reflection

Call trait methods on reflected values:

```rust
#[reflect_trait]
trait Damageable {
    fn take_damage(&mut self, amount: f32);
}

#[derive(Reflect)]
#[reflect(Damageable)]
struct Enemy {
    health: f32,
}

impl Damageable for Enemy {
    fn take_damage(&mut self, amount: f32) {
        self.health -= amount;
    }
}

fn damage_entity(entity: &mut dyn Reflect, amount: f32, registry: &TypeRegistry) {
    // Get the trait implementation
    let reflect_damageable = registry
        .get_type_data::<ReflectDamageable>(entity.type_id())
        .unwrap();
    
    // Call the trait method
    let damageable = reflect_damageable.get_mut(entity).unwrap();
    damageable.take_damage(amount);
}
```

This enables calling trait methods without knowing the concrete type at compile time.

## FromReflect

Convert reflected values back to concrete types:

```rust
#[derive(Reflect, FromReflect)]
struct Config {
    max_players: u32,
    difficulty: f32,
}

fn load_config(reflected: &dyn Reflect) -> Config {
    Config::from_reflect(reflected).unwrap()
}
```

`FromReflect` enables creating concrete instances from dynamic reflected data. This is essential for deserializing into specific types and cloning reflected values.

## Scene Integration

Bevy's scene system relies on reflection:

```rust
#[derive(Component, Reflect, Default)]
#[reflect(Component)]
struct Health {
    current: f32,
    max: f32,
}

// Register with app
app.register_type::<Health>();
```

The `#[reflect(Component)]` attribute tells the reflection system this is a component, enabling scene serialization.

## Practical Applications

**Scene serialization** - Save and load game state without manual serialization code.

**Inspector tools** - Build editor UIs that automatically show and edit component fields.

**Save systems** - Serialize player data and game state generically.

**Modding support** - Let mods register new types that integrate with existing systems.

**Animation** - Animate component properties by name rather than hardcoded field access.

**Network replication** - Serialize component changes for multiplayer games.

## Reflection Requirements

Types must meet certain criteria for reflection:

**All fields must implement Reflect** - You can't reflect a struct if one of its fields doesn't support reflection.

**Types must be owned or borrowed** - Reflection works with owned types and references, not arbitrary pointers.

**Register types** - Types must be registered with the app's type registry to enable full reflection features.

Most Rust standard library types and Bevy types already implement `Reflect`, making it easy to use.

## Limitations

Reflection has costs and constraints:

**Performance overhead** - Dynamic access is slower than direct field access. Use reflection where flexibility matters more than raw speed.

**Type erasure** - Once something becomes `&dyn Reflect`, you lose compile-time type information. Recover it with downcasting or `FromReflect`.

**Requires registration** - Types need explicit registration for full functionality. Forgetting to register a type causes runtime errors.

**Reflection bounds** - Generic types require their type parameters to implement `Reflect`, which can be limiting.

## Best Practices

**Register all reflected types** - Call `app.register_type::<T>()` for every type you want to reflect. Missing registrations cause subtle bugs.

**Use FromReflect for cloning** - When you need to create owned values from reflected data, derive `FromReflect`.

**Reflect traits sparingly** - Trait reflection is powerful but adds complexity. Use it when dynamic dispatch is genuinely needed.

**Profile reflection code** - Reflection is slower than direct access. Measure if you're using it in performance-critical paths.

**Document reflected fields** - Since reflection accesses fields by name, typos become runtime errors. Good documentation helps.

Reflection is the foundation for Bevy's most dynamic features. Understanding it unlocks powerful patterns for building flexible, data-driven games and tools.

