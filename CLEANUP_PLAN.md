# Content Cleanup Plan

## Feedback Summary

1. Remove version comparison code (before/after examples)
2. Remove version-based section headings
3. Remove deprecated content entirely
4. Organize by feature/concept, not version
5. Remove checkmarks and AI-ish formatting
6. Focus on "Why" before "How"
7. Write as cohesive book, not changelog
8. Keep only latest/greatest information

## Files to Fix

### High Priority (Core Concepts)
- [x] apps.md - PARTIALLY DONE
- [ ] ecs.md - IN PROGRESS (many version refs remain)
- [ ] assets.md
- [ ] transforms.md
- [ ] events.md
- [ ] states.md

### Medium Priority (Features)
- [ ] 2d-features.md
- [ ] 3d-features.md
- [ ] pbr.md
- [ ] ui.md
- [ ] input.md
- [ ] windows.md

### Lower Priority (Platform/Advanced)
- [ ] web-support.md
- [ ] mobile-support.md
- [ ] new-renderer.md
- [ ] render-graph.md
- [ ] properties.md (reflection)
- [ ] compile-times.md
- [ ] task-system.md
- [ ] scenes.md
- [ ] sound.md

## Approach for Each File

1. Read current content
2. Identify "Why" for each major feature
3. Remove all "Added in X" / "Changed in X" / "Before/After" comparisons
4. Keep single brief note if evolution context is valuable
5. Remove deprecated sections entirely
6. Reorganize by concept, not chronology
7. Remove checkmarks/boxes
8. Focus on current, best practices

## Example Transformations

### Before:
```
## Bevy 0.5 Improvements

**Added in Bevy 0.5**

Rich text support was added...

**Before (0.4):**
```rust
// old way
```

**After (0.5):**
```rust
// new way
```

✅ Multiple styles
✅ Different fonts
```

### After:
```
## Rich Text

Text can contain multiple sections with different styles. This is useful for highlighting, colored text, or mixing fonts within a single text element.

```rust
text: Text {
    sections: vec![
        TextSection {
            value: "Score: ".to_string(),
            style: TextStyle { ... },
        },
        TextSection {
            value: "100".to_string(),
            style: TextStyle { ... },
        },
    ],
}
```

Why multiple sections? It's more efficient than multiple Text entities and respects layout as a single block.
```

## Status

- Apps: 70% complete
- ECS: 30% complete
- Others: 0% complete

