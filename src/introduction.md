# Introduction

After months of work, Bevy Engine is here!

**Bevy is a refreshingly simple data-driven game engine and app framework built in Rust. It is free and open-source forever!**

## Design Goals

- **Capable**: Offer a complete 2D and 3D feature set
- **Simple**: Easy for newbies to pick up, but infinitely flexible for power users
- **Data Focused**: Data-oriented architecture using the Entity Component System paradigm
- **Modular**: Use only what you need. Replace what you don't like
- **Fast**: App logic should run quickly, and when possible, in parallel
- **Productive**: Changes should compile quickly ... waiting isn't fun

## Key Features

### Bevy ECS
A custom Entity Component System with unrivaled usability and blisteringly-fast performance.

### Render Graphs
Easily build your own multi-threaded render pipelines using Render Graph nodes.

### Bevy UI
A custom ECS-driven UI framework built specifically for Bevy.

### Productive Compile Times
Expect changes to compile in ~0.8-3.0 seconds with the "fast compiles" config.

## General Purpose Features

- **Cross Platform**: Windows, MacOS, and Linux (with planned support for mobile and web)
- **3D**: Lights, meshes, textures, MSAA, and GLTF loading
- **Sprites**: Render individual images as sprites, render from sprite sheets, and dynamically generate new sprite sheets
- **Assets**: An extensible, event driven asset system that loads assets asynchronously in background threads
- **Scenes**: Save ECS Worlds to human-readable scene files and load scene files into ECS Worlds
- **Plugins**: All engine and app features are implemented as modular plugins
- **Sound**: Load audio files as assets and play them from within systems
- **Multiple Render Backends**: Vulkan, DirectX 12, and Metal (with more on the way thanks to wgpu)
- **Data Driven Shaders**: Easily bind ECS components directly to shader uniforms
- **Hot Asset Reloading**: Automatically reload changes to assets at runtime without recompiles or restarts
- **Events**: Efficiently consume and produce Events from within ECS systems
- **Properties**: Dynamically get and set component fields using a string version of their names
- **Hierarchical Transforms**: Create parent-child relationships between entities that propagate Transforms down the hierarchy

## Current State

Bevy is in the "prototyping" phase: features are missing, APIs will change, and documentation is sparse. It's not yet recommended for serious projects unless you're willing to deal with gaps and instability.

## Getting Started

Check out the Quick Start Guide to dive in, or continue reading to learn about what Bevy offers today.

