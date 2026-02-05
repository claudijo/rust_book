# mdbook Theme Testing Guide

## Quick Theme Switcher

To try different themes, edit `book.toml` and change the `default-theme` value:

```toml
[output.html]
default-theme = "THEME_NAME"
```

Then rebuild: `mdbook build`

## Available Themes:

### 1. **rust** (your original)
The Rust documentation style - clean, professional, orange accents.
```toml
default-theme = "rust"
```

### 2. **ayu** (currently active)
Modern theme with great syntax highlighting, multiple color variants.
- Clean and minimal
- Good for code-heavy content
- Supports dark/light modes
```toml
default-theme = "ayu"
```

### 3. **coal**
Dark theme with high contrast - great for late-night reading.
- Dark gray/black background
- Excellent for reducing eye strain
- Good code readability
```toml
default-theme = "coal"
```

### 4. **navy**
Dark blue theme - professional and elegant.
- Deep blue background
- Good for technical documentation
- Nice color scheme variety
```toml
default-theme = "navy"
```

### 5. **light**
Minimal, clean light theme.
- Pure white background
- High contrast text
- Very traditional documentation look
```toml
default-theme = "light"
```

## Interactive Theme Switching

mdbook also allows readers to switch themes on the fly using the theme picker (paint brush icon) in the top toolbar. All themes are included by default, so readers can choose their preference.

## Recommendations:

**For code-heavy technical books:** `ayu` or `rust`  
**For dark mode preference:** `coal` or `navy`  
**For maximum simplicity:** `light`  
**For Rust ecosystem familiarity:** `rust` (your original)

## Current Status:
✅ Currently testing: **ayu**  
📖 Open `book/index.html` in your browser to see it!

To revert to rust theme:
```bash
# Edit book.toml to set default-theme = "rust"
mdbook build
```

