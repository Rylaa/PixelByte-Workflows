# Inline Text Variations Reference

> **Used by:** code-generator-swiftui, code-generator-react

## Overview
Handles text nodes with `characterStyleOverrides` that produce multi-styled text (different colors, weights within a single text element).

---

## Reading from Implementation Spec

Check for "### Inline Text Variations" section in Components:

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

## SwiftUI Code Generation

When Inline Text Variations exist, generate Text concatenation:

```swift
// Single-color text (no variations)
Text("Simple text")
  .foregroundColor(.white)

// Multi-color text (with variations from spec)
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
```

## Generation Rules

1. **Wrap in parentheses** when using + operator for Text concatenation
2. **Each Text segment** gets its own modifiers based on variation table
3. **Font applies to each segment** (cannot be applied to concatenated result)
4. **Decoration (underline/strikethrough)** applies only to relevant segment
5. **Color from hex** when variation color differs from primary

## Template

```swift
// For each variation row in table:
Text("{variation.text}")
  .font(.system(size: {fontSize}, weight: .{weight}))
  .foregroundColor({colorModifier})
  {decorationModifier}

// colorModifier:
// - #FFFFFF → .white
// - #000000 → .black
// - other → Color(hex: "{color}")

// decorationModifier:
// - underline → .underline()
// - strikethrough → .strikethrough()
// - none → (omit)
```

## Example Output

Input spec:
```markdown
| Range | Text | Color | Weight | Decoration |
|-------|------|-------|--------|------------|
| 0-15 | "Let's fix your " | #FFFFFF | 600 | none |
| 15-19 | "Hook" | #F2F20D | 600 | underline |
```

Generated SwiftUI:
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

## Common Mistakes

- Applying font to concatenated result -> Compilation error
  - Apply font to each Text segment individually

- Missing parentheses around concatenation -> Modifier scope issues
  - Wrap entire concatenation in parentheses

- Using + without parentheses as body -> Type inference issues
  - Wrap in `Group { }` or parentheses when returning from body
