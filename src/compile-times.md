# Productive Compile Times

One of Bevy's primary design goals is "productivity". Game development is an extremely iterative and experimental process full of small changes. If each change takes a significant amount of time to test, then development becomes a grind.

## Acceptability Scale for Iterative Changes

- **0-1 seconds**: ideal
- **1-3 seconds**: fine
- **3-5 seconds**: annoying
- **5-10 seconds**: painful but still usable if you are committed
- **10+ seconds**: completely unusable

> **Note**: These are "iterative compile times" not "clean compile times". Clean compiles only need to happen once, whereas iterative compiles happen constantly.

## Current Performance

Currently, with the "fast compiles" configuration, changes to Bevy examples can be compiled in ~0.8-3 seconds, based on your computer specs, configuration, and OS choice. Bevy falls into the "ideal to fine" range.

## Why Rust Compiles Slowly

Rust code generally compiles slowly for three reasons:

1. **Generic Monomorphization**: The compile step where generic code gets turned into a non-generic copy. Compile times go up as the volume of monomorphized code goes up. To keep costs low you should either avoid generics entirely or keep generic code "small" and shallow.

2. **Link Time**: How long it takes to link code. Here the important thing is to keep code volume and dependency counts low.

3. **LLVM**: Rust throws large amounts of IR code at LLVM and expects it to optimize it. This takes time. Additionally LLVM is optimized for "fast code at runtime" more than "fast code generation".

## The Challenges

The cards are stacked against game engines:

**The Game Engine Domain:**
- Game engines inherently touch a large number of domains (and therefore involve a large number of dependencies)
- Game engines are "big" - they require a lot of code

**Rust's Design Choices:**
- Dependencies are statically linked by default, which means every new dependency adds link times
- Rust's default linker is quite slow
- Cargo makes taking dependencies very easy. What appears to be a small, simple crate might actually have a large dependency tree

## Bevy's Approach

Bevy takes a pragmatic approach. Being willing to take dependencies is good for the Rust ecosystem - we don't want to ignore all the great work that has already been done, especially when it comes to projects like winit and wgpu. But we still strive to keep our dependency tree as small as we can. Any dependency that takes Bevy out of the "ideal to fine" iterative compile time range must either be pared down or removed.

## The "Fast Compiles" Configuration

The "fast compiles" configuration achieves usable iterative compile times while still taking dependencies. It consists of three parts:

1. **LLD linker**: LLD is much faster at linking than the default Rust linker. This is the biggest win.

2. **Dynamic Linking** (Bevy 0.4+): Link Bevy as a shared library for development.

3. **Incremental compilation**: Use Cargo's incremental compilation for better caching.

## Dynamic Linking

**Added in Bevy 0.4**

Dynamic linking significantly reduces iterative compile times during development:

```bash
# For your Bevy app
cargo run --features bevy/dynamic

# For Bevy examples  
cargo run --features dynamic --example my_example
```

### Performance Impact

**Example: Recompiling 3d_scene after a change**
- Static linking: ~30 seconds
- **Dynamic linking: ~4 seconds**

**7-8x faster iteration!**

### How It Works

With dynamic linking, Bevy's engine code is compiled into a shared library (`.so`, `.dylib`, `.dll`). When you change your game code:
- ✅ Only your code recompiles
- ✅ Engine code is reused
- ✅ Much faster linking

### When to Use

✅ **Use during development:**
- Rapid iteration
- Testing changes
- Daily development work

❌ **Don't use for release:**
- Publishing your game
- Final distribution
- Release builds

Disable the feature when building your final game binary!

### Trade-offs

**Pros:**
- 👍 7-8x faster iterative compiles
- 👍 Faster linking
- 👍 Quick iteration cycle

**Cons:**
- 👎 Slightly larger debug builds
- 👎 Requires managing shared libraries
- 👎 Not suitable for distribution

Combined with LLD linker and other optimizations, this keeps Bevy in the "ideal to fine" compile time range!

2. **Nightly Rust Compiler**: Gives access to the latest performance improvements and "unstable" optimizations. Note that Bevy can still be compiled on stable Rust if that is a requirement for you.

3. **Generic Sharing**: Allows crates to share monomorphized generic code instead of duplicating it. In some cases this allows us to "precompile" generic code so it doesn't affect iterative compiles.

## Setup

To enable fast compiles:
1. Install the nightly rust compiler
2. Install LLD
3. Copy the fast compiles config file to YOUR_WORKSPACE/.cargo/config

## Current Limitations

**MacOS**: doesn't have an up-to-date version of the LLD linker, so iterative compiles are slower on that platform.

**Windows**: LLD is slightly slower on Windows than on Linux.

## Future Improvements

### Dynamic Linking

An easy way to cut down on link times is to dynamically link. Bevy can support dynamic App plugins to achieve ~0.6 second iterative compile times. The new Bevy ECS currently doesn't support dynamic linking because it relies on TypeIds (which are incompatible with dynamic linking). However, this problem has been solved in other projects and will be added back soon.

### Cranelift Rustc Backend

Cranelift is an alternative compiler backend optimized for fast compiles. The rustc cranelift backend is rapidly approaching a usable state and should provide a nice boost eventually.

