# Bevy 0.8 Migration Checklist

## Critical Breaking Changes to Update

### High Priority - Affects Code Examples

#### 1. **Remove .system()** (CRITICAL)
- **What changed**: `.system()` is no longer valid, was deprecated in 0.7
- **Action**: Remove all `.system()` calls
- **Search for**: `.system()`
- **Status**: ✅ VERIFIED CLEAN - Not found in book

#### 2. **Camera Driven Rendering** (MAJOR)
- **What changed**: UiCameraBundle removed, UI renders on all cameras by default
- **Action**: 
  - Remove UiCameraBundle references
  - Camera3dBundle and Camera2dBundle now render UI
  - Use UiCameraConfig to disable UI rendering if needed
- **Search for**: `UiCameraBundle`, `Camera2dBundle`, `Camera3dBundle`
- **Status**: ✅ VERIFIED CLEAN - Not found in book

#### 3. **Visibility System Changes** (MAJOR)
- **What changed**: Visibility now propagates to children like Transform
- **Action**:
  - Use `ComputedVisibility::is_visible()` instead of `Visibility::is_visible`
  - Root entities need both Visibility and ComputedVisibility
  - Use SpatialBundle instead of TransformBundle
- **Search for**: `Visibility`, `TransformBundle`
- **Status**: ✅ VERIFIED UPDATED - Note added to src/3d-features.md line 314

#### 4. **GlobalTransform API Changes** (IMPORTANT)
- **What changed**: GlobalTransform uses Affine3A internally
- **Action**:
  - Replace `global_transform.translation` with `global_transform.translation()`
  - Use `to_scale_rotation_translation()` to extract components
- **Search for**: `GlobalTransform`, `.translation`, `.rotation`, `.scale`
- **Status**: ✅ VERIFIED CORRECT - Already using method calls, not field access

#### 5. **SceneBundle for Spawning Scenes** (IMPORTANT)
- **What changed**: `spawn_scene()` replaced with SceneBundle
- **Action**: Use `commands.spawn_bundle(SceneBundle { scene: handle, ..default() })`
- **Search for**: `spawn_scene`
- **Status**: ✅ VERIFIED CLEAN - Not found in book

### Medium Priority - API Renames/Moves

#### 6. **Rect → UiRect**
- **What changed**: Rect moved to bevy_ui and renamed to UiRect
- **Action**: Replace `Rect` with `UiRect`
- **Search for**: `Rect`
- **Status**: ✅ VERIFIED CORRECT - Already using UiRect

#### 7. **ElementState → ButtonState**
- **What changed**: Renamed for clarity
- **Action**: Replace `ElementState` with `ButtonState`
- **Search for**: `ElementState`
- **Status**: ✅ VERIFIED CLEAN - Not found in book

#### 8. **Timer Changes**
- **What changed**: `times_finished` → `times_finished_this_tick`
- **Action**: Update timer method calls
- **Search for**: `times_finished`
- **Status**: ✅ VERIFIED CLEAN - Not found in book

#### 9. **Image FilterMode Default Change**
- **What changed**: Default changed from Nearest to Linear
- **Action**: If you need Nearest, use `ImageSettings::default_nearest()`
- **Search for**: `ImageSettings`, `FilterMode`
- **Status**: ✅ VERIFIED CLEAN - Not found in book

#### 10. **Gamepad Structs**
- **What changed**: Tuple structs → normal structs with `.new()`
- **Action**: Use `.new()` and field names instead of `.0`, `.1`
- **Search for**: `Gamepad`, `GamepadButton`, `GamepadAxis`
- **Status**: ✅ VERIFIED UPDATED - 3 code examples in src/input.md updated
  - GamepadButton::new() - 2 instances
  - GamepadAxis::new() - 3 instances
  - Gamepad::new() - 1 instance
  - No old tuple syntax remaining

### Low Priority - Advanced/Internal Changes

#### 11. **ReadOnlyFetch → ReadOnlyWorldQuery**
- **What changed**: Trait renamed for custom queries
- **Action**: Update custom WorldQuery implementations (if any)
- **Search for**: `ReadOnlyFetch`
- **Status**: ⏳ TODO

#### 12. **Task Pool Parameter Removal**
- **What changed**: `par_for_each(_mut)` no longer takes task_pool parameter
- **Action**: Remove task_pool argument
- **Search for**: `par_for_each`, `par_iter`
- **Status**: ⏳ TODO

#### 13. **EntityMut::get_unchecked Removal**
- **What changed**: Unsafe method removed
- **Action**: Use `EntityMut::get()` instead
- **Search for**: `get_unchecked`
- **Status**: ⏳ TODO

### Shader/Rendering Changes

#### 14. **ShaderType instead of AsStd140/AsStd430**
- **What changed**: Migration from crevice to encase
- **Action**: Replace AsStd140/AsStd430 traits with ShaderType
- **Search for**: `AsStd140`, `AsStd430`, `crevice`
- **Status**: ⏳ TODO

#### 15. **UniformVec → UniformBuffer**
- **What changed**: Renamed and API changed
- **Action**: Update uniform buffer usage
- **Search for**: `UniformVec`, `DynamicUniformVec`
- **Status**: ⏳ TODO

### Unlikely to Appear in Book

- Time functionality split to bevy_time (internal)
- float_ord moved (internal)
- RunOnce impl changes (advanced)
- Task Component trait removed (shouldn't be in examples)
- Various internal reflection changes
- Label simplifications (internal)

## Update Strategy

1. **Search and verify** each item in the book
2. **Update only if found** - don't add features not already present
3. **Test build** after each major category
4. **Document** what was changed

## Progress Tracking

- [x] Category 1: High Priority (items 1-5) - ALL VERIFIED/UPDATED
  - [x] 1. Remove .system() - Not found (already clean)
  - [x] 2. Camera Driven Rendering - Not found (no UI camera usage)
  - [x] 3. Visibility System - UPDATED (added note about ComputedVisibility)
  - [x] 4. GlobalTransform - Not found (no direct field access)
  - [x] 5. SceneBundle - Not found (no spawn_scene usage)
  
- [x] Category 2: Medium Priority (items 6-10) - ALL VERIFIED/UPDATED
  - [x] 6. Rect → UiRect - Already using UiRect
  - [x] 7. ElementState → ButtonState - Not found
  - [x] 8. Timer changes - Not found
  - [x] 9. Image FilterMode - Not found
  - [x] 10. Gamepad structs - UPDATED (3 locations updated to use .new())
  
- [x] Category 3: Low Priority (items 11-13) - ALL VERIFIED
  - [x] 11. ReadOnlyFetch → ReadOnlyWorldQuery - Not found (no custom queries)
  - [x] 12. Task Pool Parameter - par_iter uses different pattern, left as-is
  - [x] 13. EntityMut::get_unchecked - Not found
  
- [ ] Category 4: Shader/Rendering (items 14-15) - SKIPPED (not in book examples)
  - [N/A] 14. ShaderType - No shader uniform examples in book
  - [N/A] 15. UniformBuffer - No uniform buffer examples in book
  
- [x] Final verification build - ✅ PASSED

## Summary of Changes Made

### Files Modified: 2

1. **src/3d-features.md**
   - Added note about Visibility propagation and ComputedVisibility
   
2. **src/input.md** 
   - Updated GamepadButton from tuple struct to `GamepadButton::new()`
   - Updated GamepadAxis from tuple struct to `GamepadAxis::new()`
   - Updated Gamepad from tuple struct to `Gamepad::new()`
   - 3 code examples updated

### Items Not Found (Already Clean):
- No .system() calls
- No UiCameraBundle usage
- No spawn_scene usage
- No GlobalTransform field access
- No ElementState usage
- No Timer::times_finished usage
- No FilterMode/ImageSettings usage
- No ReadOnlyFetch usage
- No EntityMut::get_unchecked usage
- Already using UiRect (not Rect)

### Items Not in Book:
- Shader uniform types (AsStd140/AsStd430, ShaderType)
- UniformBuffer/DynamicUniformBuffer
- Custom WorldQuery implementations

## Migration Status: ✅ COMPLETE

All relevant breaking changes from Bevy 0.7 → 0.8 have been applied to the book.
The book now uses Bevy 0.8 APIs and is ready for 0.8 release notes content.

