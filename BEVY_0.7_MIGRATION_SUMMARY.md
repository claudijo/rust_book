# Bevy 0.7 Migration Summary

## Updates Applied to the Book

### ECS Chapter (`src/ecs.md`)
**1. ParamSet replaces QuerySet**
- Updated "Conflicting Queries" section
- Changed `QuerySet` → `ParamSet`
- Changed `.q0()/.q1()` → `.p0()/.p1()`
- Example now shows modern ParamSet API

**2. Resource Initialization**
- Added "Resource Initialization" section under Commands
- Documented `commands.init_resource::<T>()` convenience method
- Shows how it initializes resources with Default trait

### Windows Chapter (`src/windows.md`)
**1. PresentMode replaces VSync**
- Updated "VSync" section → "Present Mode"
- Changed `window.set_vsync(bool)` → `window.set_present_mode(PresentMode)`
- Documented all PresentMode variants:
  - `Fifo` (VSync enabled)
  - `Immediate` (no VSync)
  - `Mailbox` (low latency with VSync)
  - `AutoVsync` / `AutoNoVsync`
- Updated example in "Best Practices" section

### Verified As Already Correct
The following chapters were verified to already use Bevy 0.7+ APIs:

**3D Features Chapter:**
- Already uses `mesh.insert_attribute()` (not deprecated `set_attribute`)
- Already uses `Mesh::ATTRIBUTE_*` constants
- Camera examples don't use deprecated `camera.name` strings

**UI Chapter:**
- No usage of deprecated `Margins` type
- Already uses `Rect` where needed

**Transforms Chapter:**
- No usage of deprecated `Mat4::face_toward()`
- Already uses `Mat4::look_at_rh()`

**Events Chapter:**
- Uses `bevy::prelude::*` which includes correct event types
- No direct imports that would be affected by module moves

### Deprecated APIs Not Found
The following deprecated APIs were verified to NOT exist in the book:
- ✓ No `.into_system()` usage
- ✓ No `.config()` API usage
- ✓ No `RunSystem` usage
- ✓ No `camera.name` usage
- ✓ No `World::entities_mut()` usage (would need unsafe in 0.7)
- ✓ No deprecated `Margins` type usage

## Changes Summary

### Files Modified: 3
1. `src/ecs.md` - 2 updates
2. `src/windows.md` - 2 updates
3. `BEVY_0.7_MIGRATION_CHECKLIST.md` - tracking document

### Files Verified (No Changes Needed): 16
- `src/3d-features.md`
- `src/2d-features.md`
- `src/apps.md`
- `src/assets.md`
- `src/audio.md`
- `src/compile-times.md`
- `src/events.md`
- `src/input.md`
- `src/mobile-support.md`
- `src/pbr.md`
- `src/reflection.md`
- `src/scenes.md`
- `src/states.md`
- `src/task-system.md`
- `src/transforms.md`
- `src/ui.md`
- `src/web-support.md`

## Migration Guide Coverage: 100%

All breaking changes from the Bevy 0.6 → 0.7 migration guide have been addressed:
1. ✅ ParamSet for conflicting SystemParam
2. ✅ AliasedMutability (QueryEntityError update - no examples to update)
3. ✅ Remove margins.rs (not present in book)
4. ✅ Remove face_toward.rs (not present in book)
5. ✅ World::entities_mut is now unsafe (not used in book examples)
6. ✅ Custom vertex attributes (already using correct API)
7. ✅ Mesh vertex buffer layouts (already using correct API)
8. ✅ Remove .into_system() (not present in book)
9. ✅ Remove RunSystem (not present in book)
10. ✅ Replace VSync with PresentMode (updated)
11. ✅ Fix mul_vec3 transformation order (no code changes needed)
12. ✅ Camera marker components (already using correct API)
13. ✅ Remove the config API (not present in book)
14. ✅ Cameras point at RenderTarget (already using correct API)
15. ✅ Implement init_resource (documented)
16. ✅ Infallible resource getters (examples already use Res<T> in systems)
17. ✅ Event types moved (using prelude, not affected)

## Book Status: Ready for 0.7 Release Notes

The migration guide updates are complete. The book now:
- Uses all Bevy 0.7 APIs correctly
- Contains no deprecated 0.6 code
- Is ready to receive additional content from 0.7 release notes

Next step: Add new features from Bevy 0.7 release notes.

