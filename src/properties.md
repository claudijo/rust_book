# Reflection

Bevy provides runtime reflection capabilities to interact with Rust types dynamically.

## History

**Bevy 0.1-0.3: Properties System**

Early Bevy versions used `bevy_property` and `bevy_type_registry` crates. While functional, they were:
- Custom-tailored to Bevy's needs
- Full of custom jargon
- Couldn't handle traits
- Had fundamental restrictions on data access

**Bevy 0.4+: Bevy Reflect**

The new `bevy_reflect` crate replaced the old system. It's designed to be a generic Rust reflection library useful beyond just Bevy.

## Basic Usage

Derive the `Reflect` trait to enable reflection:

```rust
#[derive(Reflect)]
struct Foo {
    a: u32,
    b: Vec<Bar>,
    c: Vec<u32>,
}

#[derive(Reflect)]
struct Bar {
    value: String
}

let mut foo = Foo {
    a: 1,
    b: vec![Bar { value: "hello world".to_string() }],
    c: vec![1, 2]
};
```

## Field Access by Name

```rust
// Get field value
assert_eq!(*foo.get_field::<u32>("a").unwrap(), 1);

// Set field value
*foo.get_field_mut::<u32>("a").unwrap() = 2;
assert_eq!(foo.a, 2);
```

## Patching with New Values

```rust
let mut dynamic_struct = DynamicStruct::default();
dynamic_struct.insert("a", 42u32);
dynamic_struct.insert("c", vec![3, 4, 5]);

foo.apply(&dynamic_struct);

assert_eq!(foo.a, 42);
assert_eq!(foo.c, vec![3, 4, 5]);
```

## Nested Field Access

Look up nested fields using "path strings":

```rust
let value = *foo.get_path::<String>("b[0].value").unwrap();
assert_eq!(value.as_str(), "hello world");
```

This is powerful for editor tools and debugging!

## Iterating Over Fields

```rust
for (i, value: &Reflect) in foo.iter_fields().enumerate() {
    let field_name = foo.name_at(i).unwrap();
    if let Ok(value) = value.downcast_ref::<u32>() {
        println!("{} is a u32 with value: {}", field_name, *value);
    }
}
```

## Automatic Serialization

Serialize and deserialize without manual Serde implementations:

```rust
// Setup type registry
let mut registry = TypeRegistry::default();
registry.register::<u32>();
registry.register::<String>();
registry.register::<Bar>();
registry.register::<Foo>();

// Serialize
let serializer = ReflectSerializer::new(&foo, &registry);
let serialized = ron::ser::to_string_pretty(&serializer, Default::default()).unwrap();

// Deserialize
let mut deserializer = ron::de::Deserializer::from_str(&serialized).unwrap();
let reflect_deserializer = ReflectDeserializer::new(&registry);
let value = reflect_deserializer.deserialize(&mut deserializer).unwrap();
let dynamic_struct = value.take::<DynamicStruct>().unwrap();

// Reflect has its own partial_eq
assert!(foo.reflect_partial_eq(&dynamic_struct).unwrap());
```

## Trait Reflection

**Added in Bevy 0.4**

Call traits on `&dyn Reflect` references without knowing the underlying type:

```rust
#[derive(Reflect)]
#[reflect(DoThing)]
struct MyType {
    value: String,
}

impl DoThing for MyType {
    fn do_thing(&self) -> String {
        format!("{} World!", self.value)
    }
}

#[reflect_trait]
pub trait DoThing {
    fn do_thing(&self) -> String;
}

// Box as dyn Reflect
let reflect_value: Box<dyn Reflect> = Box::new(MyType {
    value: "Hello".to_string(),
});

// Setup registry
let mut type_registry = TypeRegistry::default();
type_registry.register::<MyType>();

// Get the reflected trait
let reflect_do_thing = type_registry
    .get_type_data::<ReflectDoThing>(reflect_value.type_id())
    .unwrap();

// Convert &dyn Reflect to &dyn DoThing
let my_trait: &dyn DoThing = reflect_do_thing.get(&*reflect_value).unwrap();

// Call the trait method - magic!
println!("{}", my_trait.do_thing()); // "Hello World!"
```

This enables calling traits on reflected types, which is useful for editor tools and plugin systems.

## FromReflect Trait

**Added in Bevy 0.6**

The `FromReflect` trait enables creating "clones" of types using arbitrary Reflect implementations:

```rust
#[derive(Reflect, FromReflect)]
struct Foo {
    bar: usize,
    baz: String,
}

// Create a concrete type from a reflected value
let reflected: &dyn Reflect = &some_reflected_value;
let foo = Foo::from_reflect(reflected).unwrap();
```

### Why FromReflect?

Before FromReflect, you couldn't easily convert a `&dyn Reflect` back into a concrete type. This made it difficult to:
- Clone reflected values
- Deserialize into concrete types
- Work with reflected collections properly

**Use cases:**
- Making reflected collections (like `Vec<T>`) work properly
- "Round trip" conversions to and from Reflect types
- Cloning dynamically typed values
- Deserialization into concrete types

**Example with collections:**
```rust
#[derive(Reflect, FromReflect)]
struct Inventory {
    items: Vec<Item>,
}

#[derive(Reflect, FromReflect)]
struct Item {
    name: String,
    quantity: usize,
}

// Reflect can now properly clone the entire structure
let reflected_inventory: &dyn Reflect = &inventory;
let cloned = Inventory::from_reflect(reflected_inventory).unwrap();
```

This is especially important for scene serialization and editor tooling!

## Use Cases

Reflection powers many Bevy features:

### Scene System
Scenes use reflection for:
- Serialization/deserialization
- Component patching
- Hot reloading

### Future: Editor
Planned uses include:
- Inspector widgets (automatically generated)
- Undo/redo systems
- Property animation

### Future: Animation
Property-based animation will use reflection to animate component fields dynamically.

## Requirements

To use reflection:
- Derive `Reflect` on your types
- All fields must implement `Reflect`
- Most core Rust and Bevy types already implement Reflect

## Bevy Reflect vs Properties

**Improvements in 0.4:**
- ✅ Generic Rust reflection (not Bevy-specific)
- ✅ Reflects Rust language constructs directly
- ✅ Handles traits (trait reflection)
- ✅ More flexible data access
- ✅ Better serialization support
- ✅ Clearer terminology

The new system is more powerful and easier to use, while being general enough for non-Bevy projects.

## Legacy: Properties (0.1-0.3)

The old Properties system had similar goals but different implementation. If you're upgrading:

**Old (Properties):**
```rust
#[derive(Properties)]
struct Counter {
    count: u32,
}

counter.set_prop_val::<u32>("count", 2);
```

**New (Reflect):**
```rust
#[derive(Reflect)]
struct Counter {
    count: u32,
}

*counter.get_field_mut::<u32>("count").unwrap() = 2;
```

The concepts are similar, but Reflect is more powerful and standard.

