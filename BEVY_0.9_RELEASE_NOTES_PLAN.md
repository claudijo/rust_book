# Bevy 0.9 Release Notes - Content Integration Plan

## Topics from Table of Contents

### Post Processing & Rendering
1. HDR Post Processing, Tonemapping, and Bloom
2. FXAA: Fast Approximate Anti-Aliasing
3. Deband Dithering
4. Post Processing: View Target Double Buffering
5. Improved Render Target Texture Format Handling

### Scenes
6. New Scene Format
7. Binary Scene Formats
8. Dynamic Scene Builder
9. More Scene Construction Tools

### ECS Improvements
10. Improved Entity / Component APIs
11. Exclusive System Rework
12. Bevy ECS Now Uses GATS!
13. Derive Resource
14. System Ambiguity Resolution API Improvements
15. Bevy ECS Optimizations
16. ECS Change Detection Bypass

### Reflection
17. Enum Reflection
18. Other Bevy Reflect Improvements

### Input
19. Example: Gamepad Viewer
20. Axis and Button Settings Validation
21. ScanCode Input Resource

### Rendering Features
22. Time Shader Globals
23. High Entity Renderer Slowdown Optimization
24. Vertex Attributes Fully Optional
25. Expose Multi Draw Indirect
26. KTX2 Array / Cubemap / Cubemap Array Textures
27. Camera::viewport_to_world
28. Multiple Directional Lights
29. Sprite Rects

### Plugins & Architecture
30. Plugin Settings
31. Plugins Are Now Unique By Default

### Task System
32. Task Pool: Nested Spawns on Scope
33. Task Pool Panic Handling

### Hierarchy
34. Hierarchy Query Methods

### UI
35. Bevy UI: The Origin Is Now In The Top Left (Already migrated!)
36. Bevy UI: Z-Indices

## Integration Strategy

Since I can't download the full HTML easily right now, I'll work with what we know:

### Phase 1: Post Processing & Rendering (High Priority)
- Add HDR, tonemapping, bloom to 3D features
- Add FXAA to rendering
- Note other rendering improvements

### Phase 2: Scenes (High Priority)
- Update scenes chapter with new format
- Binary scenes
- Scene builder APIs

### Phase 3: ECS (Medium Priority - Many Already Covered in Migration)
- Verify all ECS items already updated
- Add any new ECS patterns not in migration

### Phase 4: Reflection (Medium Priority)
- Enum reflection (important!)
- Other reflection improvements

### Phase 5: Input, Rendering Details, UI (Lower Priority)
- Input improvements
- Rendering optimizations (mostly internal)
- UI Z-indices

## Next Steps

1. Fetch release notes content piece by piece if needed
2. Start with high-priority rendering features
3. Update systematically through each phase
4. Verify book builds after each phase

Ready to proceed once we can access the full content!

