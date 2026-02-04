# Properties

Bevy Properties add some dynamism to Rust, which is a notoriously static language. It is often useful to get or set a struct's field using a string version of its name, or interact with a struct when you don't have a statically typed reference. Languages generally cover these cases with "reflection" features, but unfortunately Rust does not currently have this type of reflection. The bevy_property crate provides a subset of useful "reflection-like" features in Rust.

## Basic Usage

```rust
#[derive(Properties)]
pub struct Counter {
    count: u32,
}

let mut counter = Counter { count: 1 };

// You can set a property value like this. The type must match exactly or this will fail.
counter.set_prop_val::<u32>("count", 2);
assert_eq!(counter.count, 2);
assert_eq!(counter.prop_val::<u32>("count").unwrap(), 2);

// You can also set properties dynamically. set_prop accepts any type that implements
// the Property trait, but the property type must match the field type or this
// operation will fail.
let new_count: u32 = 3;
counter.set_prop("count", &new_count);
assert_eq!(counter.count, 3);
```

## DynamicProperties

DynamicProperties also implements the Properties trait, but it has no restrictions on field names or types:

```rust
let mut patch = DynamicProperties::map();
patch.set_prop_val::<usize>("count", 4);

// You can "apply" Properties on top of other Properties. This will only set
// properties with the same name and type. You can use this to "patch" your
// properties with new values.
counter.apply(&patch);
assert_eq!(counter.count, 4);

// Types that implement Properties can be converted to DynamicProperties
let dynamic_thing: DynamicProperties = counter.to_dynamic();
```

## Use Cases

Properties are what make Bevy's Scene system so nice to use. They're also planned for use in the upcoming Bevy Editor, such as:

- Undo/redo
- Viewing and editing component properties at runtime
- Property animation tools

## Serialization

Types that implement Properties can be serialized using serde and DynamicProperties can be deserialized using serde. When combined with the Properties patching feature, this means any type that derives Properties can be round trip serialized and deserialized.

## Requirements

To derive Properties each field in the struct must implement the Property trait. This is already implemented for most core Rust and Bevy types, so you should only need to implement Property for custom types (and you can derive Property too).

> **Note**: The bevy_property crate may be useful in non-Bevy contexts and will be published to crates.io in the near future.

