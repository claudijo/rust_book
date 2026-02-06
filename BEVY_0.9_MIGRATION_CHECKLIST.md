# Bevy 0.9 Migration Guide - Complete Checklist

## Overview
This migration guide has **62 migration topics** - significantly larger than 0.7→0.8 (which had ~15). 
Many breaking changes across multiple systems including Resources, ECS, UI, Input, and more.

## Complete Migration Topics (62 total)

### Resources & Plugins
1. **Make Resource trait opt-in, requiring #[derive(Resource)] V2** - CRITICAL
2. **Plugins own their settings. Rework PluginGroup trait** - Plugin API changes
3. **Use plugin setup for resource only used at setup time** - Resource optimization
4. **Remove AssetServer::watch_for_changes()** - Asset API change

### ECS Core Changes
5. **Spawn now takes a Bundle** - CRITICAL API change
6. **Accept Bundles for insert and remove. Deprecate insert/remove_bundle** - CRITICAL
7. **Implement Bundle for Component. Use Bundle tuples for insertion** - Important

### Time & Timers
8. **Replace the bool argument of Timer with TimerMode** - Timer API change
9. **Add global time scaling** - New feature
10. **Add TimeUpdateStrategy resource for manual Time updating** - Advanced

### UI System (MAJOR CHANGES)
11. **Change UI coordinate system to have origin at top left corner** - BREAKING
12. **Rename UiColor to BackgroundColor** - Breaking rename
13. **Make the default background color of NodeBundle transparent** - Behavior change
14. **Clarify bevy::ui::Node field and documentation** - Documentation
15. **Remove Size and UiRect generics** - Type simplification
16. **Remove margins.rs** - API removal
17. **Move Size to bevy_ui** - Module reorganization
18. **Move Rect to bevy_ui and rename it to UiRect** - Already done in 0.8?
19. **Move sprite::Rect into bevy_math** - Module reorganization

### Systems
20. **Exclusive Systems Now Implement System. Flexible Exclusive System Params** - Advanced
21. **Rename system chaining to system piping** - Terminology

### Input & Gamepad
22. **Change gamepad.rs tuples to normal structs** - Already done in 0.8?
23. **Add getters and setters for InputAxis and ButtonSettings** - API addition
24. **Add GamepadInfo, expose gamepad names** - New feature
25. **Gamepad type is Copy; do not require / return references** - API change

### Rendering & Windows
26. **Update wgpu to 0.14.0** - Backend update
27. **Support monitor selection for all window modes** - New feature
28. **Make raw_window_handle field in Window and ExtractedWindow an Option** - API change

### Transforms
29. **Add associated constant IDENTITY to Transform and friends** - Convenience
30. **Rename Transform::mul_vec3 to transform_point and improve docs** - Breaking rename
31. **Remove Transform::apply_non_uniform_scale** - API removal
32. **Remove face_toward.rs** - API removal

### Advanced ECS
33. **Replace WorldQueryGats trait with actual gats** - Internal
34. **Add a method for accessing the width of a Table** - Internal
35. **Make Handle<T> field id private, and replace with a getter** - API change
36. **Add methods for silencing system-order ambiguity warnings** - Advanced

### Miscellaneous
37. **Remove unused DepthCalculation enum** - Cleanup
38. **Fix inconsistent children removal behavior** - Behavior fix
39. **Entity's "ID" should be named "index" instead** - Naming
40. **Remove ExactSizeIterator from QueryCombinationIter** - API change
41. **Query filter types must be ReadOnlyWorldQuery** - Type requirement

### TextureAtlas & Animation
42. **Merge TextureAtlas::from_grid_with_padding into TextureAtlas::from_grid** - API merge
43. **Rename play to start and add new play method** - Animation API

### Reflection
44. **Remove ReflectMut in favor of Mut<dyn Reflect>** - Reflection API
45. **remove blanket Serialize + Deserialize requirement for Reflect** - Trait bounds
46. **Derive Reflect + FromReflect for input types** - More types reflected
47. **Relax bounds on Option<T>** - Type flexibility

### Scene & Serialization
48. **Scene serialization format improvements** - Internal

### Diagnostics
49. **Changed diagnostics from seconds to milliseconds** - Unit change
50. **Add Exponential Moving Average into diagnostics** - New feature

### Other
51. **Nested spawns on scope** - Task system
52. **Extract Resources into their own dedicated storage** - Internal
53. **Clean up Fetch code** - Internal
54. **Rename ElementState to ButtonState** - Already done in 0.8
55. **Fix incorrect and unnecessary normal-mapping code** - Bug fix
56. **Utility methods for Val** - UI convenience
57. **Allow passing glam vector types as vertex attributes** - Rendering
58. **Add pop method for List trait** - Reflection API
59. **Remove an outdated workaround for impl Trait** - Internal
60. **Add a change detection bypass and manual control over change ticks** - Advanced
61. **Make internal struct ShaderData non-pub** - Internal
62. **Make Children constructor pub(crate)** - Internal
63. **Rename Handle::as_weak() to cast_weak()** - API rename
64. **Remove Sync bound from Local** - Type constraint change
65. **Add FromWorld bound to T in Local<T>** - Type constraint addition
66. **Swap out num_cpus for std::thread::available_parallelism** - Internal

## Missing From Initial List (Now Added)
- Add pop method for List trait
- Remove an outdated workaround for impl Trait
- Add a change detection bypass and manual control
- Make internal struct ShaderData non-pub
- Make Children constructor pub(crate)
- Rename Handle::as_weak() to cast_weak()
- Remove Sync bound from Local
- Add FromWorld bound to T in Local<T>
- Swap out num_cpus

## Total: 66 migration topics (not 57!)

## Priority Classification

### CRITICAL (Must Update - Will Break Book)
- Resource trait requires #[derive(Resource)]
- Spawn now takes a Bundle
- insert/remove_bundle deprecated
- UI coordinate system origin change (top-left)
- UiColor → BackgroundColor
- Transform::mul_vec3 → transform_point
- Timer bool → TimerMode

### HIGH (Likely Used in Book)
- Bundle for Component
- UI Size and UiRect changes
- Gamepad API changes (if already updated in 0.8)
- Animation play → start

### MEDIUM (May Be Used)
- AssetServer::watch_for_changes removal
- Plugin settings changes
- Handle<T> API changes
- Children removal behavior

### LOW (Internal/Advanced)
- WorldQueryGats
- Exclusive System params
- Resource storage extraction
- Fetch code cleanup

## Estimated Impact
- **Files likely affected:** 10-15 chapters
- **Critical updates:** ~7-10 major API changes
- **Estimated changes:** 50-100 locations

## Strategy
1. Start with CRITICAL items first (Resource, Spawn, UI)
2. Then HIGH priority items
3. Verify build after each category
4. Document all changes
5. Create verification checklist

Ready to begin systematic migration!

