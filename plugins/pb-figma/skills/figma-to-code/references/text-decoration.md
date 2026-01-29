# Text Decoration Reference

> **Used by:** design-analyst, code-generator-*

Guide for extracting text decoration properties from Figma TEXT nodes and mapping them to CSS/Tailwind and SwiftUI code.

---

## Figma textDecoration Values

| Value | Description |
|-------|-------------|
| `NONE` | No decoration |
| `UNDERLINE` | Underline beneath text |
| `STRIKETHROUGH` | Line through the middle of text |

---

## Extraction from Figma TEXT Nodes

### Detection Criteria

Check the `textDecoration` property on TEXT nodes via `figma_get_node_details`.

### REST API Limitation

- Figma REST API only provides the basic `textDecoration` type (NONE, UNDERLINE, STRIKETHROUGH)
- Advanced properties (decoration color, thickness) are Plugin API only, not available in REST API
- Use the text node's fill color as the decoration color

### Extraction Process

For each decorated text node:

1. **Extract decoration properties:**

   ```
   figma_get_node_details:
     - file_key: {file_key}
     - node_id: {text_node_id}

   Read from response:
     - textDecoration: UNDERLINE | STRIKETHROUGH
     - fills: [{ hex: "#ffd100", opacity: 1.0 }]  # Use for decoration color
   ```

2. **Decoration color = text fill color:**

   ```
   Text fills[0].hex: "#ffd100" with opacity: 1.0
   -> Use this color for text decoration
   ```

### Implementation Spec Output Format

Only add this section if `textDecoration` is not NONE:

```markdown
### Text Decoration

**Component:** {ComponentName}
- **Decoration:** Underline | Strikethrough
- **Color:** {text_fill_color} (uses text color)

**Tailwind Mapping:** `underline decoration-[{color}]`
**SwiftUI Mapping:** `.underline(color: Color(hex: "{color}"))` or `.strikethrough(color: Color(hex: "{color}"))`
```

**Rules:**
- Only add this section if text has decoration (`textDecoration` is not NONE)
- Decoration color must match text fill color (REST API limitation)
- Omit thickness property (not available in REST API)

---

## Inline Text Variations (Mixed Decorations)

A single TEXT node may have multiple character styles, including different decorations for different words.

### Detection via Figma API

Check for `characterStyleOverrides` array on TEXT nodes. When overrides exist:

1. Extract each style from `styleOverrideTable`
2. Read per-style: fills, `textDecoration`
3. Document character ranges and their styles
4. If `styleOverrideTable` contains empty objects `{}` for an override index, treat those characters as using the node's base style

### Design-Validator Extraction

```javascript
const decoration = style.textDecoration || 'NONE';
// Returns: "fills: #ffd100, decoration: UNDERLINE"
```

### Spec Output for Inline Variations

```markdown
### Inline Text Variations

**Component:** TitleText
**Full Text:** "Let's fix your Hook"
**Variations:**
| Range | Text | Color | Weight | Decoration |
|-------|------|-------|--------|------------|
| 0-15 | "Let's fix your " | #FFFFFF | 600 | none |
| 15-19 | "Hook" | #F2F20D | 600 | underline |
```

---

## CSS / Tailwind Mapping

### Basic Decoration

```tsx
// Underline with custom color
<span className="underline decoration-[#ffd100]">
  Underlined Text
</span>

// Strikethrough with color
<span className="line-through decoration-[#ff0000]/80">
  Strikethrough Text
</span>

// Combined with other text styles
<span className="text-lg font-semibold underline decoration-primary decoration-2">
  Styled Underline
</span>
```

### Tailwind Decoration Properties

| Property | Tailwind Class | Example |
|----------|----------------|---------|
| Underline | `underline` | `underline` |
| Strikethrough | `line-through` | `line-through` |
| Color | `decoration-{color}` | `decoration-red-500` |
| Arbitrary color | `decoration-[#hex]` | `decoration-[#ffd100]` |
| Thickness | `decoration-{n}` | `decoration-2` |
| Opacity | `decoration-{color}/{opacity}` | `decoration-red-500/50` |

### CSS Fallback (Exact Control)

```tsx
<span
  style={{
    textDecoration: 'underline',
    textDecorationColor: '#ffd100',
    textDecorationThickness: '2px'
  }}
>
  Underlined
</span>
```

---

## SwiftUI Mapping

### Basic Decoration

```swift
// Underline with custom color (iOS 16+)
@available(iOS 16.0, *)
struct HookText: View {
  var body: some View {
    Text("Hook")
      .font(.system(size: 14, weight: .regular))
      .underline(color: Color(hex: "#ffd100"))
  }
}

// Strikethrough with color and opacity (iOS 16+)
@available(iOS 16.0, *)
struct StrikeText: View {
  var body: some View {
    Text("Strike")
      .font(.system(size: 14, weight: .regular))
      .strikethrough(color: Color(hex: "#ff0000").opacity(0.8))
  }
}

// Basic underline (no custom color, all iOS versions)
struct BasicText: View {
  var body: some View {
    Text("Basic")
      .font(.system(size: 14, weight: .regular))
      .underline()
  }
}
```

### iOS Version Considerations for Decoration Color

The `color:` parameter on `.underline()` and `.strikethrough()` requires iOS 16+.

**Option 1: iOS 16+ only (recommended for new apps):**

```swift
@available(iOS 16.0, *)
struct HookText: View {
  var body: some View {
    Text("Hook")
      .underline(color: Color(hex: "#ffd100"))
  }
}
```

**Option 2: With iOS 15 fallback (backward compatibility):**

```swift
struct HookText: View {
  var body: some View {
    if #available(iOS 16.0, *) {
      Text("Hook")
        .font(.system(size: 14, weight: .regular))
        .underline(color: Color(hex: "#ffd100"))
    } else {
      Text("Hook")
        .font(.system(size: 14, weight: .regular))
        .underline()  // No color on iOS 15
    }
  }
}
```

### Decoration Modifier Rules

1. Apply after `.font()` modifier -- decoration goes after typography, never before
2. Use exact color from spec -- copy hex value from the "Color" field
3. Include opacity if < 1.0 -- add `.opacity(0.8)` to Color when decoration color has opacity < 1.0
4. iOS 16+ API -- add `@available(iOS 16.0, *)` when using the `color:` parameter
5. Fallback for iOS 15 -- use `.underline()` or `.strikethrough()` without color for older iOS versions

---

## Multi-Segment Text (Inline Variations)

### SwiftUI: Text Concatenation

When inline variations are detected, generate Text concatenation with the `+` operator:

```swift
private var titleText: some View {
  (
    Text("Let's fix your ")
      .font(.system(size: 24, weight: .semibold))
      .foregroundColor(.white)
    +
    Text("Hook")
      .font(.system(size: 24, weight: .semibold))
      .foregroundColor(Color(hex: "#F2F20D"))
      .underline()
  )
}
```

**Generation rules:**

1. Wrap in parentheses when using `+` operator for Text concatenation
2. Each Text segment gets its own modifiers based on the variation table
3. Font applies to each segment individually (cannot be applied to the concatenated result)
4. Decoration (underline/strikethrough) applies only to the relevant segment
5. Use Color from hex when the variation color differs from the primary text color

**Template per variation row:**

```swift
Text("{variation.text}")
  .font(.system(size: {fontSize}, weight: .{weight}))
  .foregroundColor({colorModifier})
  {decorationModifier}

// colorModifier:
// - #FFFFFF -> .white
// - #000000 -> .black
// - other   -> Color(hex: "{color}")

// decorationModifier:
// - underline      -> .underline()
// - strikethrough  -> .strikethrough()
// - none           -> (omit)
```

### CSS / Tailwind: Span Segments

For React, wrap each variation segment in a `<span>` with appropriate classes:

```tsx
<p className="text-2xl font-semibold">
  <span className="text-white">Let's fix your </span>
  <span className="text-[#F2F20D] underline">Hook</span>
</p>
```

---

## Common Mistakes and Best Practices

### CSS / Tailwind

| Mistake | Correct Approach |
|---------|-----------------|
| `text-decoration: underline #ffd100` (invalid shorthand) | `underline decoration-[#ffd100]` (Tailwind pattern) |
| Missing decoration color when spec has it | Copy exact color from spec: `decoration-[#ffd100]` |

### SwiftUI

| Mistake | Correct Approach |
|---------|-----------------|
| `.underline()` before `.font()` | `.font().underline()` -- typography first, then decoration |
| `.underline(color: .yellow)` (system color) | `.underline(color: Color(hex: "#ffd100"))` -- exact spec color |
| `.underline(color: Color(hex: "#ff0000"))` when opacity is 0.8 | `.underline(color: Color(hex: "#ff0000").opacity(0.8))` -- include opacity |
| Using `color:` parameter without `@available(iOS 16.0, *)` | Add `@available(iOS 16.0, *)` to the struct -- prevents compilation error on iOS 15 |
| `.underline(color: Color(hex: "#ffd100").opacity(1.0))` | `.underline(color: Color(hex: "#ffd100"))` -- no `.opacity()` when value is 1.0 |
| Applying `.font()` to concatenated Text result | Apply `.font()` to each Text segment individually -- compilation error otherwise |
| Missing parentheses around `+` concatenation | Wrap entire concatenation in parentheses -- prevents modifier scope issues |
| Using `+` without parentheses as view body | Wrap in `Group { }` or parentheses when returning from body -- avoids type inference issues |
