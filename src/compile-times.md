# Productive Compile Times

Game development demands rapid iteration. Change code, test it, change again. If compilation takes too long, this creative flow breaks down. Bevy prioritizes fast iterative compilation to keep you productive.

## The Iteration Speed Problem

How long can you wait before testing changes?

- **0-1 seconds** - Feels instant, maintains flow state
- **1-3 seconds** - Noticeable but acceptable
- **3-5 seconds** - Starts to break concentration
- **5-10 seconds** - Enough time to lose your train of thought
- **10+ seconds** - Completely disrupts workflow

These thresholds are for iterative compiles - recompiling after small changes. Clean builds from scratch can take longer since they happen rarely.

Bevy aims for the 0-3 second range with proper configuration.

## Why Rust Compiles Slowly

Rust's design choices prioritize runtime performance over compile speed:

**Monomorphization** - Generic code generates specialized versions for each type used. More generics mean more code to compile.

**Link time** - Combining compiled code into an executable takes time, especially with many dependencies.

**LLVM optimization** - The compiler spends time optimizing code for maximum runtime performance.

These choices make Rust programs fast to run but slow to compile.

## Game Engine Challenges

Game engines face particular compilation difficulties:

**Broad scope** - Engines touch graphics, audio, physics, networking, and more. Each domain adds dependencies.

**Large codebases** - Comprehensive engines require substantial code.

**Rich type systems** - Complex generic APIs provide flexibility but increase compilation work.

**Static linking** - Dependencies link into your binary, each one adding to link time.

Rust makes dependencies easy to add, but each one carries compilation cost.

## Bevy's Strategy

Bevy uses several techniques to maintain fast iteration:

### Dynamic Linking

Enable dynamic linking during development:

```bash
cargo run --features bevy/dynamic
```

This compiles Bevy as a shared library. When you change your game code, only your code recompiles - the engine library is reused.

**Impact:**
- Normal iteration: 20-30 seconds
- With dynamic linking: 3-4 seconds
- **7-8x faster**

Use dynamic linking for daily development. Disable it for release builds - shipping games should link statically for better performance and simpler distribution.

### Fast Linker

The default Rust linker is slow. LLD (LLVM's linker) is much faster.

**On Linux:**
```bash
sudo apt install lld
```

**On Windows:**
LLD comes with Rust.

**Configure in `.cargo/config.toml`:**
```toml
[target.x86_64-unknown-linux-gnu]
linker = "clang"
rustflags = ["-C", "link-arg=-fuse-ld=lld"]

[target.x86_64-pc-windows-msvc]
linker = "rust-lld.exe"
```

This single change significantly reduces link time.

### Incremental Compilation

Cargo's incremental compilation reuses work from previous builds:

```toml
[profile.dev]
incremental = true
```

This is enabled by default in debug builds. It caches compilation results, making subsequent builds faster.

### Optimized Build Profiles

Configure different optimization levels for dependencies vs your code:

```toml
[profile.dev.package."*"]
opt-level = 3

[profile.dev]
opt-level = 1
```

This compiles dependencies with full optimization (once) but your code with minimal optimization (fast to compile). Your code iterates quickly while libraries remain performant.

## Complete Configuration

Create `.cargo/config.toml` in your project:

```toml
[target.x86_64-unknown-linux-gnu]
linker = "clang"
rustflags = ["-C", "link-arg=-fuse-ld=lld"]

[target.x86_64-pc-windows-msvc]
linker = "rust-lld.exe"

[target.x86_64-pc-windows-gnu]
linker = "rust-lld.exe"
rustflags = ["-C", "link-arg=-fuse-ld=lld"]

[target.x86_64-apple-darwin]
rustflags = ["-C", "link-arg=-fuse-ld=lld"]
```

Add to `Cargo.toml`:

```toml
[profile.dev]
opt-level = 1

[profile.dev.package."*"]
opt-level = 3
```

With these settings and dynamic linking, Bevy achieves 1-4 second iterative compile times on most machines.

## Platform Differences

**Linux** - Best compile times. LLD works excellently.

**Windows** - Good compile times. LLD is built-in but slightly slower than on Linux.

**macOS** - Slower than Linux/Windows. macOS's default linker is less optimized, and LLD integration is less mature.

## Development Workflow

Optimize your workflow around fast compilation:

**Use dynamic linking during development:**
```bash
cargo run --features bevy/dynamic
```

**Disable it for release builds:**
```bash
cargo build --release
```

**Use `cargo check` for syntax verification:**
```bash
cargo check
```

This validates code without generating executables - much faster when you just want to verify changes compile.

**Keep your dependency tree lean** - Every dependency adds compilation time. Evaluate whether new dependencies are worth their cost.

## Measuring Compile Time

Track compilation performance:

```bash
cargo clean
cargo build --timings
```

This generates a report showing where compilation time goes. Use it to identify slow dependencies or bottlenecks.

## Future Improvements

Rust compilation continues to get faster:

**Cranelift backend** - Alternative compiler backend optimized for compilation speed over runtime performance. Good for debug builds.

**Parallel front-end** - The Rust compiler is becoming more parallel, utilizing multiple CPU cores better.

**Better caching** - Improved incremental compilation reduces redundant work.

Bevy tracks these developments and adopts improvements as they mature.

## Best Practices

**Enable dynamic linking** - Use it for all development work.

**Configure fast linking** - Set up LLD on your platform.

**Optimize dependency builds** - Let Cargo optimize dependencies once while keeping your code fast to compile.

**Use cargo check** - Verify code without full compilation when possible.

**Profile occasionally** - Run `cargo build --timings` periodically to catch compilation regressions.

**Keep dependencies minimal** - Each crate adds to build time. Only add dependencies that provide significant value.

Fast iteration is essential for productive game development. Bevy's compilation optimizations keep you focused on creating rather than waiting.

