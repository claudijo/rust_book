# Bevy 0.7 Migration Tracking

## Changes from Migration Guide 0.6 → 0.7

### ECS Chapter Updates Needed:
1. **ParamSet replaces QuerySet** - Update conflicting queries section
   - `QuerySet` → `ParamSet`
   - `.q0()` → `.p0()`, `.q1()` → `.p1()`
   
2. **World::entities_mut is now unsafe** - Update world access documentation

3. **Infallible resource getters** - New API for resources
   - `world.get_resource::<T>().unwrap()` → `world.resource::<T>()`
   - Add new `.resource()` method documentation

4. **World::insert_non_send renamed** - Update resource insertion
   - `world.insert_non_send()` → `world.insert_non_send_resource()`

5. **Event types moved** - Update event imports
   - `bevy::app::{Events, EventReader, EventWriter}` → `bevy::ecs::event::{...}`

6. **Remove .into_system()** - Simplify run criteria examples
   - No longer needed when piping run criteria

7. **Remove .config() API** - Update system configuration examples
   - Show closure-based approach instead

8. **init_resource for Commands and World** - Document new convenience methods

### 3D/Rendering Chapter Updates:
9. **Custom vertex attributes** - Update mesh customization
   - `mesh.set_attribute()` → `mesh.insert_attribute()`
   - Use `MeshVertexAttribute` instead of strings
   - Show proper attribute ID generation

10. **Mesh vertex buffer layouts** - Simplified API
    - Remove manual stride/offset calculations
    - Use `VertexBufferLayout::from_vertex_formats()`

11. **Camera marker components** - Replace name strings
    - Remove `Camera.name` examples
    - Use marker components with `With<T>` queries
    - Update camera spawn examples

12. **Cameras point at RenderTarget** - Update camera targeting
    - `Camera.window` → `Camera.target: RenderTarget::Window(id)`
    - Support rendering to textures

### Windows Chapter Updates:
13. **PresentMode replaces VSync** - Update window configuration
    - `WindowDescriptor.vsync: bool` → `WindowDescriptor.present_mode: PresentMode`
    - Document PresentMode variants: Immediate, Mailbox, Fifo, etc.

### Transforms Chapter Updates:
14. **Fix mul_vec3 transformation order** - Note correct behavior
    - Transforms now consistently: scale → rotate → translate
    - SpriteBundle behaves as expected when rotating

### UI Chapter Updates:
15. **Remove margins.rs** - Update UI types
    - `Margins` → `Rect`

### Removed/Deprecated:
16. **Remove face_toward.rs** - Update transform utilities
    - `Mat4::face_toward()` → `Mat4::look_at_rh()`

17. **Remove RunSystem** - Remove any examples using this

## Update Order:
1. ECS (multiple changes)
2. Windows (PresentMode)
3. 3D Features (cameras, meshes)
4. Transforms (transformation order note)
5. UI (Margins → Rect)

## Status:
- [x] ECS chapter - ParamSet updated ✓
- [x] ECS chapter - init_resource added ✓
- [x] Windows chapter - PresentMode updated ✓
- [x] 3D Features chapter - Already using correct API ✓
- [x] Transforms chapter - No face_toward to update ✓
- [x] UI chapter - No Margins to update ✓

## Remaining to verify:
- [x] Check for any .into_system() usage - None found ✓
- [x] Check for .config() API usage - None found ✓
- [x] Check for RunSystem usage - None found ✓
- [x] Verify all event import examples are correct - Using prelude ✓
- [x] Check for any World::entities_mut() usage - None found ✓
- [x] Check for any old camera.name usage - None found ✓

## Migration Complete! ✓

All code examples in the book have been verified to be compatible with Bevy 0.7:
- ParamSet API updated
- PresentMode replaces VSync
- init_resource documented
- No deprecated APIs present
- All mesh and camera APIs already using 0.7+ syntax

The book is now ready for Bevy 0.7 release notes content!

