# Content Guidelines for Bevy Programming Guide

## Core Principles

### 1. Focus on Current, Best Practices
- Document only the latest API and approaches
- Remove all deprecated content
- No "before/after" version comparisons
- Keep content evergreen and immediately useful

### 2. Organize by Concept, Not Version
- Group content by feature/functionality
- Avoid version-based section headings (e.g., "Bevy 0.X Improvements")
- Structure follows logical learning paths, not chronological development

### 3. Lead with "Why" Before "How"
- Explain the purpose and motivation before implementation
- Help readers understand when and why to use features
- Provide context for design decisions
- Integrate motivation naturally, avoid explicit "Why X?" headings
- Use descriptive headings like "Understanding X", "The Problem with X", "Design Rationale"

### 4. Professional Tone
- Avoid AI-generated style markers (checkmarks, bullet emojis, "exciting features!")
- Write like a technical book, not a blog or changelog
- Be clear, direct, and informative

### 5. Brief Historical Context (When Valuable)
- A single, brief note about evolution is acceptable if it adds understanding
- Example: "Transform uses a similarity-based approach (translation + rotation + scale) for numerical stability"
- Don't include: version numbers, dates, multiple migration paths

## What to Exclude

### Version References
❌ "Added in Bevy 0.X"
❌ "Changed in Bevy 0.X"
❌ "New in Bevy 0.X"
❌ Section headings like "Bevy 0.X Improvements"

### Code Comparisons
❌ Before/After code blocks showing old vs new APIs
❌ Multiple code examples for different versions
❌ Migration examples

Exception: Keep one brief note if it clarifies why something works the way it does.

### Deprecated Content
❌ Sections about removed features
❌ Old APIs that no longer work
❌ Workarounds for old limitations

### Formatting
❌ Checkmarks (✅/❌)
❌ Emoji bullets
❌ "Pros/Cons" lists with icons
❌ Boxes and special formatting that screams "AI generated"

## What to Keep and Emphasize

### Conceptual Organization
✓ Group related functionality together
✓ Clear section hierarchies
✓ Logical progression of complexity

### Explanatory Content
✓ Why this feature exists
✓ What problems it solves
✓ When to use it vs alternatives
✓ Trade-offs and considerations

### Practical Examples
✓ Complete, working code examples
✓ Real-world use cases
✓ Common patterns
✓ Best practices

### Technical Details
✓ How features work internally (when relevant)
✓ Performance characteristics
✓ Memory layout implications
✓ Safety guarantees

## Content Structure Template

### Chapter Structure
```
# Feature Name

Brief intro - what is this feature?

## Why [Feature Name]?

Explain the problem this solves and motivation for its design.

## Basic Usage

Show the simplest, most common use case with complete example.

## Key Concepts

Explain the main ideas and components.

## Common Patterns

Show typical ways to use this feature.

## Advanced Topics

Cover edge cases, optimizations, or complex scenarios.

## Best Practices

Guidance on effective usage.
```

### Example Section (Good)
```markdown
## Change Detection

Change detection lets you react only when components are modified, avoiding unnecessary work. Many systems only need to run when data actually changes - a UI system might only update when health changes, not every frame.

```rust
fn react_to_damage(query: Query<&Health, Changed<Health>>) {
    for health in query.iter() {
        println!("Health changed to {}", health.current);
    }
}
```

The `Changed<T>` filter matches entities where component T was added or modified this frame.
```

### Example Section (Avoid)
```markdown
## Why Change Detection?

Change detection is important because...

## Change Detection

✅ Added in Bevy 0.5
```

### Example Section (Bad)
```markdown
## Change Detection

✅ Added in Bevy 0.5

Change detection was significantly improved!

**Before (0.4):**
```rust
// old way
```

**After (0.5):**
```rust
// new way
```

Benefits:
✅ More efficient
✅ Easier to use
✅ Better performance
```

## Code Style

### Examples Should Be:
- Complete (can copy-paste and run)
- Minimal (only show relevant parts)
- Clear (with comments explaining non-obvious aspects)
- Practical (solve real problems)

### Good Example
```rust
fn spawn_enemy(mut commands: Commands) {
    commands.spawn((
        Enemy,
        Health { current: 100.0, max: 100.0 },
        Transform::from_xyz(10.0, 0.0, 0.0),
    ));
}
```

### Bad Example
```rust
// Bevy 0.5+ way (much better than 0.4!)
fn spawn_enemy(mut commands: Commands) { // Note: Commands changed in 0.5
    // Using the new spawn API
    commands.spawn((  // ✅ New in 0.5!
        Enemy,
        Health { current: 100.0, max: 100.0 },
        Transform::from_xyz(10.0, 0.0, 0.0),
    ));
}
```

## Exceptions

### When Version Context Is Acceptable

Brief mentions are okay when they:
1. Clarify a design decision
2. Help understand API choices
3. Provide valuable context

Example (acceptable):
"Bevy's Transform was redesigned to use similarity transforms (translation + rotation + scale) rather than raw matrices, providing better numerical stability and simpler APIs."

Example (not acceptable):
"In Bevy 0.3, Transform was redesigned. Before 0.3 it used... After 0.3 it uses..."

## File Handling Safety

**Critical:** When cleaning up chapters, files can easily become empty or corrupted. Follow this process strictly.

### Safe File Replacement Process

1. **Create the cleaned content in a NEW file**
   ```
   Create: chapter_cleaned.md
   ```

2. **Verify the new file has content**
   ```bash
   head -20 chapter_cleaned.md
   wc -l chapter_cleaned.md
   ```

3. **Use cat with redirect to copy (most reliable)**
   ```bash
   cat chapter_cleaned.md > chapter.md
   ```
   
   **DO NOT use:**
   - `mv` (can fail silently)
   - `cp` without verification (can fail silently)

4. **Verify the target file has content**
   ```bash
   head -20 chapter.md
   wc -l chapter.md
   ```

5. **Only after verification, remove temporary file**
   ```bash
   rm chapter_cleaned.md
   ```

### Verification Checklist

After every file replacement:
- [ ] Check file is not empty (`wc -l filename.md`)
- [ ] Check file has expected content (`head -20 filename.md`)
- [ ] Verify line count is reasonable (not 0, not 1)
- [ ] If empty, restore from `_cleaned` version immediately

### If a File Becomes Empty

1. **Stop immediately** - Don't continue with other files
2. **Check for `_cleaned` version** - It should still exist
3. **Restore using cat redirect**:
   ```bash
   cat chapter_cleaned.md > chapter.md
   ```
4. **Verify restoration** before continuing
5. **Investigate why it happened** to prevent recurrence

### Prevention

- Always work with temporary `_cleaned` files first
- Never delete temporary files until target is verified
- Use `cat file > target` instead of `mv` or `cp`
- Always verify after each operation
- If multiple chapters need cleaning, do them one at a time with verification between each

### Terminal Command Safety

**Prefer:**
```bash
# Explicit and verifiable
cat source.md > target.md
head -10 target.md  # Verify immediately
```

**Avoid:**
```bash
# Can fail silently
mv source.md target.md
cp source.md target.md
```

## Quality Checklist

Before considering a chapter complete:

- [ ] No version numbers in headings
- [ ] No before/after code comparisons
- [ ] No deprecated content
- [ ] No checkmarks or emoji formatting
- [ ] Organized by concept, not chronology
- [ ] "Why" explained before "How"
- [ ] Examples are complete and practical
- [ ] Technical terms defined on first use
- [ ] Trade-offs and considerations mentioned
- [ ] Reads like a professional programming book

## Maintenance

When adding new content from version releases:

1. **Extract the concepts**, not the changelog
2. **Integrate into existing structure** by concept
3. **Update existing sections** with new information
4. **Remove superseded content** entirely
5. **Rewrite** rather than append version-specific sections

The goal is a cohesive book that reads as if written at one moment in time, not a layered history of changes.

## Completeness Verification

**Critical:** When integrating a new version's content, ensure ALL planned additions are completed before moving on.

### Process for New Versions

1. **Read entire release notes** - Don't skim, read every section
2. **Create feature list** - Document every feature mentioned
3. **Plan integration** - Note which chapter each feature belongs in
4. **Track progress** - Mark each feature as added
5. **Verify completion** - Cross-check the list before declaring done
6. **Never leave placeholders** - No "Content coming soon" sections

### Feature Checklist Template

When starting a new version integration, create a checklist:

```markdown
## Bevy X.Y Feature Integration

### Features from Release Notes:
- [ ] Feature 1 - Target chapter: ECS
- [ ] Feature 2 - Target chapter: Rendering
- [ ] Feature 3 - Target chapter: Input
...

### Sections Updated:
- [ ] Chapter 1 - specific changes
- [ ] Chapter 2 - specific changes
...

### Verification:
- [ ] All features from release notes documented
- [ ] No features marked "coming soon"
- [ ] All code examples tested/verified
- [ ] Guidelines followed for all additions
```

### Common Pitfalls to Avoid

**Incomplete coverage** - Don't add major features and skip smaller ones. If it's in the release notes, it should be in the book (or explicitly decided to skip with reason).

**Lost track of progress** - Without a checklist, it's easy to think you've covered everything when you haven't. Always work from a written list.

**Premature completion** - Don't declare a version "done" until you've verified every item on your checklist is addressed.

### How to Verify Completeness

Before declaring a version integrated:

1. **Re-read the table of contents** from the release notes
2. **Search the book** for each feature by name
3. **Check your initial feature list** - every item should have a checkmark
4. **Review chapters** that were supposed to be updated
5. **Ask yourself**: "If someone asked about feature X from this version, would they find it in the book?"

If any feature is missing or has a placeholder, the integration is not complete.

### Documentation Over Speed

It's better to:
- Take longer and be complete
- Ask for checkpoint reviews after each section
- Maintain a detailed checklist
- Verify each addition

Than to:
- Rush through and miss features
- Skip verification steps
- Leave incomplete sections
- Have to go back and fill gaps later

## Examples of Well-Structured Content

See the following chapters as references:
- `apps.md` - Clean organization, clear purpose
- `ecs.md` - Strong "Why" sections, concept-based structure
- `states.md` - Practical examples, clear use cases

These demonstrate the style and structure we're aiming for throughout the book.

