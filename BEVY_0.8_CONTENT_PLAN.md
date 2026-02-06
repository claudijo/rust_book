# Bevy 0.8 Release Notes - Content Integration Plan

## Integration Strategy

Following the verified approach from 0.7:
1. **ONLY add content explicitly mentioned in release notes**
2. **NO hallucinations** - verify every API mentioned
3. **Integrate into existing structure** where logical
4. **Create new sections** only when topic doesn't fit existing chapters
5. **Ask for approval** before each major section

---

## Content Mapping by Category

### RENDERING & 3D (Goes to 3D Features Chapter)

#### High Priority - Core Features
1. **Camera-Driven Rendering** → 3D Features (Cameras section)
   - Render to Textures
   - Split Screen
   - Layered Rendering
   - New Camera Bundles
   - Enabling/Disabling Cameras
   - **Decision**: Expand existing Cameras section significantly

2. **Spotlights** → 3D Features (Lighting section)
   - New light type with cone shape
   - **Decision**: Add to existing Lighting section

3. **RenderLayers** → 3D Features (new subsection under Cameras or Lighting)
   - Control what cameras render
   - **Decision**: Add near camera rendering section

4. **Vertex Colors** → 3D Features (Meshes section)
   - Per-vertex color support
   - **Decision**: Add to existing mesh documentation

5. **Automatic Mesh Tangent Generation** → 3D Features (Meshes section)
   - Mention briefly
   - **Decision**: Brief note in mesh section

6. **Default to Linear Texture Filtering** → Assets Chapter
   - Already migrated, just note the behavior
   - **Decision**: Brief mention in compressed textures section

#### Medium Priority - Advanced Features
7. **Custom Render Graphs** → 3D Features (advanced section)
   - Too advanced for current scope?
   - **Decision**: SKIP or very brief mention

8. **Scene Bundle** → Assets or Scenes Chapter
   - Already migrated
   - **Decision**: Verify it's documented

9. **Parallel Frustum Culling** → 3D Features (Performance subsection)
   - Internal optimization
   - **Decision**: Brief mention in performance tips

10. **Regular Polygon and Circle Mesh Primitives** → 3D Features (Meshes)
    - New mesh creation helpers
    - **Decision**: Add examples to meshes section

### ECS IMPROVEMENTS (Goes to ECS Chapter)

11. **Query IntoIter** → ECS (Queries section)
    - Iterator improvements
    - **Decision**: Add to query iteration examples

12. **Query::iter_many** → ECS (Queries section)
    - New query method
    - **Decision**: Add alongside Query::many

13. **Convert Mutable Queries to Read-only** → ECS (Queries section)
    - Safety feature
    - **Decision**: Add to query patterns

14. **ECS "lifetimed pointers"** → ECS (advanced)
    - Advanced internal feature
    - **Decision**: SKIP (too internal)

15. **Scripting and Modding Progress: Untyped ECS APIs** → ECS (advanced)
    - World API extensions
    - **Decision**: SKIP or very brief (too advanced)

16. **ECS Query Internals Refactors** → SKIP
    - Internal only

17. **ECS Optimizations** → SKIP
    - Internal only

18. **Label Optimizations** → SKIP
    - Internal only

19. **Hierarchy Commands** → ECS (Commands section)
    - New hierarchy manipulation methods
    - **Decision**: Add to commands section

20. **ECS Soundness / Correctness Improvements** → SKIP
    - Internal improvements

### REFLECTION (Goes to Reflection Chapter)

21. **Bevy Reflection Improvements** → Reflection Chapter
    - Default Trait Reflection
    - Array Reflection
    - Static TypeInfo
    - Resource Reflection
    - Pretty Debug Formatting
    - **Decision**: Expand existing Reflection chapter

### MATERIALS & SHADERS (Goes to 3D Features or new Materials section?)

22. **New Material System** → 3D Features or NEW Chapter
    - Material trait
    - AsBindGroup derive
    - Custom shaders easier
    - **Decision**: Major feature - expand PBR chapter or create materials section

23. **Built-in Shader Modularization** → 3D Features (advanced)
    - Shader imports
    - PBR shader logic reuse
    - **Decision**: Brief mention, links to examples

24. **wgpu 0.13: New WGSL Shader Syntax** → 3D Features
    - Note about syntax changes
    - **Decision**: Brief note for shader users

25. **ShaderType derive** → SKIP
    - Already covered in migration (internal shader types)

26. **Render Phase Sorting Optimization** → SKIP
    - Internal optimization

### VISIBILITY SYSTEM (Goes to 3D Features)

27. **Visibility Inheritance** → 3D Features (Visibility section)
    - Already noted in migration
    - **Decision**: Verify expanded coverage

28. **SpatialBundle and VisibilityBundle** → ECS or 3D Features
    - Convenience bundles
    - **Decision**: Brief mention where bundles are discussed

### TRANSFORMS (Goes to Transforms Chapter)

29. **New GlobalTransform Matrix Representation** → Transforms
    - Already migrated (Affine3A)
    - **Decision**: Verify documented

### RENDERING INTERNALS (Mostly SKIP)

30. **Render World Extract** → SKIP
    - Internal rendering detail

31. **ExtractResource Trait and Plugin** → SKIP
    - Internal rendering detail

### UI (Goes to UI Chapter)

32. **Taffy migration: a refreshed UI layout library** → UI Chapter
    - Internal layout engine change
    - **Decision**: Brief note if any API changes

### PLATFORM SUPPORT (Goes to Mobile/Web chapters)

33. **Android Progress** → Mobile Support
    - Update status
    - **Decision**: Brief update to mobile chapter

### META (SKIP - No code content)

34-40. Examples, Website, CI, Org Changes → SKIP

---

## Proposed Integration Order

### Phase 1: Camera & Rendering (Highest Impact)
- Camera-Driven Rendering (major expansion)
- Spotlights
- RenderLayers
- **Approval needed after this phase**

### Phase 2: ECS Improvements
- Query::iter_many
- Query IntoIter
- Convert queries to read-only
- Hierarchy Commands
- **Approval needed after this phase**

### Phase 3: Materials & Meshes
- New Material System
- Vertex Colors
- Mesh primitives
- **Approval needed after this phase**

### Phase 4: Reflection & Other
- Reflection improvements
- Visibility inheritance (verify)
- Minor updates
- **Approval needed after this phase**

---

## Questions for User

1. **Materials**: Create new "Materials" chapter or expand existing PBR chapter?
2. **Level of detail**: Include advanced rendering topics or keep focused on essentials?
3. **Reflection**: Significantly expand reflection chapter or keep brief?

---

## Total Estimated Additions

- **New sections:** ~15-20
- **Updated sections:** ~8-10
- **Chapters affected:** 5-7
- **Estimated new content:** ~1,500-2,000 lines

**Ready to proceed with Phase 1 (Cameras & Rendering)?**

