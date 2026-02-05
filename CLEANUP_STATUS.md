# Content Cleanup Status

## Work Completed

### Apps Chapter ✅ DONE
- Removed all version comparisons (before/after code)
- Removed version-based headings ("Bevy 0.X improvements")
- Focused on current API only
- Reorganized by concept (cross-platform support, plugins, plugin groups)
- Added "Why" context for features
- Removed checkmarks

### ECS Chapter ⚠️ PARTIALLY DONE
- Rewrote introduction focusing on "Why ECS"
- Removed deprecated "for each systems" section
- Cleaned up first half of queries section
- Still needs work on:
  - Remove remaining version references throughout
  - Clean up duplicate content  
  - Remove "Added in X" annotations
  - Remove QuerySets section (largely superseded)
  - Clean up change detection section
  - Remove performance benchmark sections with version comparisons

## Remaining Work - Critical

The following chapters are the HIGHEST priority and need the same treatment:

### States Chapter (261 lines)
- Remove version comparison code
- Remove "Bevy 0.4" / "Bevy 0.5" section headings
- Focus on current stack-based state machine only
- Add "Why states?" section
- Remove checkmarks

### Events Chapter (89 lines) 
- Small file, easy to clean
- Remove before/after examples
- Focus on EventReader/EventWriter (current API)

### Transforms Chapter (190 lines)
- Remove evolution history (0.1 -> 0.2 -> 0.3)
- Focus on current similarity-based transform
- Keep one brief note about why it's designed this way

## Remaining Work - Important

### Assets Chapter (183 lines)
- Remove version annotations
- Focus on current asset system
- Organize by concept (loading, hot reloading, custom assets)

### UI Chapter (378 lines)
- Large file with many version references
- Remove "Bevy 0.X improvements" headings
- Focus on current Text, rich text, layouts

### Input Chapter (212 lines)
- Remove version references
- Focus on current input APIs
- Organize by input type (keyboard, mouse, gamepad, touch)

### 2D/3D Features
- Clean up version references
- Focus on current capabilities

### Rendering Chapters
- PBR (248 lines)
- New Renderer (313 lines)
- Render Graph (121 lines)
- These need cleaning but are lower priority

### Platform Chapters
- Web Support (96 lines)
- Mobile Support (113 lines)
- Windows (208 lines)

## Systematic Approach Needed

For each remaining chapter:

1. **Read entire chapter**
2. **Identify main concepts/features**
3. **Remove all version comparisons**
   - Delete "Before X" / "After X" code blocks
   - Keep only current implementation
4. **Remove version-based headings**
   - "Bevy 0.X Improvements" → feature name
   - "Added in Bevy X" annotations → remove or single brief note if context helps
5. **Remove deprecated content entirely**
   - No sections about removed features
6. **Reorganize by concept**
   - Group related functionality
   - Lead with "Why" before "How"
7. **Remove checkmarks and AI formatting**
8. **Add context where valuable**
   - Brief note about design decisions
   - Trade-offs
   - When to use what

## Recommendation

**Before proceeding to Bevy 0.7**, I should complete this cleanup:

1. Finish ECS chapter (largest, most important)
2. Clean States, Events, Transforms (critical concepts)
3. Clean Assets, Input, UI (frequently used)
4. Document remaining chapters as "needs cleanup"

This ensures the foundation is solid before adding more content.

## Estimate

- ECS chapter: 1-2 hours to fully clean
- States, Events, Transforms: 30-45 min each
- Assets, Input, UI: 30-45 min each
- Other chapters: 15-30 min each

Total: 6-8 hours of focused cleanup work for all chapters.

## Alternative Approach

Given time constraints, we could:
1. Complete cleanup of core chapters (Apps ✅, ECS, States, Events)
2. Document that other chapters need similar treatment
3. Apply cleanup incrementally as we add new content from 0.7+

This ensures the most-read chapters are polished while not blocking progress.

