# What is Bevy?

Bevy is a refreshingly simple data-driven game engine and app framework built in Rust. It is free and open-source forever!

## Design Goals

Bevy prioritizes several key principles:

- **Capable**: Offer a complete 2D and 3D feature set
- **Simple**: Easy for newbies to pick up, but infinitely flexible for power users
- **Data Focused**: Data-oriented architecture using the Entity Component System paradigm
- **Modular**: Use only what you need. Replace what you don't like
- **Fast**: App logic should run quickly, and when possible, in parallel
- **Productive**: Changes should compile quickly ... waiting isn't fun

## Key Features

### Bevy ECS
A custom Entity Component System with excellent usability and high performance.

### Flexible Rendering
Build custom render pipelines using render graphs, or use the built-in PBR renderer for 3D and sprite rendering for 2D.

### Bevy UI
A custom ECS-driven UI framework built specifically for Bevy.

### Productive Compile Times
With proper configuration, expect iterative changes to compile in 1-4 seconds.

## General Purpose Features

- **Cross Platform**: Windows, macOS, Linux, with support for mobile (iOS, Android) and web (WebAssembly)
- **3D Rendering**: PBR materials, lighting with shadows, meshes, animations, and GLTF loading
- **2D Rendering**: Sprite rendering, sprite sheets, texture atlases, and flexible anchoring
- **Assets**: Extensible asset system with async loading and hot reloading
- **Scenes**: Save and load ECS worlds as human-readable scene files
- **Plugins**: All engine and app features are modular plugins
- **Audio**: Asset-based audio playback with volume and speed control
- **Animation**: Skeletal animation with GLTF import support
- **Multiple Render Backends**: Vulkan, DirectX 12, Metal, and WebGL2
- **Reflection**: Runtime type information for serialization and dynamic access
- **Events**: Efficient event system integrated with ECS
- **Hierarchical Transforms**: Parent-child entity relationships with transform propagation

## Current State

Bevy is under active development. While production-ready for many use cases, be aware that:

- APIs continue to evolve between versions
- Some advanced features are still in development
- Migration guides help transition between versions
- The community is growing and supportive

## Learning Path

This book focuses on practical usage and patterns. Each chapter covers a specific area of Bevy, building from fundamentals to advanced topics. Continue reading to learn how to use Bevy effectively.

