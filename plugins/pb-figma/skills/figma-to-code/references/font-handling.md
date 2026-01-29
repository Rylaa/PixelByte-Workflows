# Font Handling Reference

## Font Detection from Figma

Typography tokens are extracted from Figma via `figma_get_design_tokens`:

```typescript
figma_get_design_tokens({
  file_key: "{file_key}",
  include_typography: true
});
```

Returned token properties:

| Property | Description | Example |
|----------|-------------|---------|
| `fontFamily` | Font family name | `"Inter"` |
| `fontPostScriptName` | PostScript name (precise variant) | `"Inter-SemiBold"` |
| `fontWeight` | Numeric weight | `600` |
| `fontSize` | Size in pixels | `24` |
| `lineHeight` | Line height (px or %) | `130%` or `31.2` |
| `letterSpacing` | Letter spacing | `-0.02em` |
| `fontStyle` | Style variant | `"normal"` or `"italic"` |

## Font Weight Mapping Table

| Figma Weight | Name | CSS | Tailwind | SwiftUI | Kotlin |
|-------------|------|-----|----------|---------|--------|
| 100 | Thin | `font-weight: 100` | `font-thin` | `.ultraLight` | `FontWeight.Thin` |
| 200 | ExtraLight | `font-weight: 200` | `font-extralight` | `.thin` | `FontWeight.ExtraLight` |
| 300 | Light | `font-weight: 300` | `font-light` | `.light` | `FontWeight.Light` |
| 400 | Regular | `font-weight: 400` | `font-normal` | `.regular` | `FontWeight.Normal` |
| 500 | Medium | `font-weight: 500` | `font-medium` | `.medium` | `FontWeight.Medium` |
| 600 | SemiBold | `font-weight: 600` | `font-semibold` | `.semibold` | `FontWeight.SemiBold` |
| 700 | Bold | `font-weight: 700` | `font-bold` | `.bold` | `FontWeight.Bold` |
| 800 | ExtraBold | `font-weight: 800` | `font-extrabold` | `.heavy` | `FontWeight.ExtraBold` |
| 900 | Black | `font-weight: 900` | `font-black` | `.black` | `FontWeight.Black` |

## Font Style

| Figma Style | CSS | Tailwind | SwiftUI |
|-------------|-----|----------|---------|
| normal | `font-style: normal` | `not-italic` | Default (no modifier) |
| italic | `font-style: italic` | `italic` | `.italic()` |

## Font Size and Line Height Extraction

```typescript
const fontSize = nodeDetails.style?.fontSize;       // e.g., 24
const lineHeightPx = nodeDetails.style?.lineHeightPx;
const lineHeight = lineHeightPx || (fontSize * 1.2); // fallback: 1.2x font size
```

**Figma to CSS/Tailwind mapping:**

| Figma | CSS | Tailwind |
|-------|-----|----------|
| Size: 12px | `font-size: 0.75rem` | `text-xs` |
| Size: 14px | `font-size: 0.875rem` | `text-sm` |
| Size: 16px | `font-size: 1rem` | `text-base` |
| Size: 18px | `font-size: 1.125rem` | `text-lg` |
| Size: 20px | `font-size: 1.25rem` | `text-xl` |
| Size: 24px | `font-size: 1.5rem` | `text-2xl` |
| Size: 30px | `font-size: 1.875rem` | `text-3xl` |
| Size: 36px | `font-size: 2.25rem` | `text-4xl` |
| Non-standard | `font-size: Xpx` | `text-[Xpx]` |

**Line height:**

| Figma | CSS | Tailwind |
|-------|-----|----------|
| 100% | `line-height: 1` | `leading-none` |
| 130% | `line-height: 1.3` | `leading-[1.3]` |
| 150% | `line-height: 1.5` | `leading-normal` |
| Fixed px | `line-height: Xpx` | `leading-[Xpx]` |

**Letter spacing:**

| Figma | CSS | Tailwind |
|-------|-----|----------|
| 0 | `letter-spacing: 0` | `tracking-normal` |
| -20 (Figma unit) | `letter-spacing: -0.02em` | `tracking-[-0.02em]` |
| Positive value | `letter-spacing: Xem` | `tracking-[Xem]` |

## Platform-Specific Font Usage

### CSS

```css
/* @font-face declaration */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-SemiBold.woff2') format('woff2'),
       url('/fonts/Inter-SemiBold.ttf') format('truetype');
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}

/* Usage */
.heading {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  font-weight: 600;
  font-size: 1.5rem;
  line-height: 1.3;
  letter-spacing: -0.02em;
}
```

### Tailwind CSS

```tsx
// Using Tailwind utility classes
<h2 className="font-['Inter'] font-semibold text-2xl leading-[1.3] tracking-[-0.02em]">
  Heading Text
</h2>

// With configured font (tailwind.config.js)
<h2 className="font-inter font-semibold text-2xl leading-[1.3]">
  Heading Text
</h2>
```

**Figma to Tailwind shorthand:**

```
Font: Inter     --> font-['Inter'] or font-inter (if configured)
Size: 24px      --> text-2xl
Weight: 600     --> font-semibold
Line Height: 130% --> leading-[1.3]
Letter Spacing: -20 --> tracking-[-0.02em]
```

### SwiftUI

```swift
// System font with weight
.font(.system(size: 24, weight: .semibold))

// Custom font
.font(.custom("Inter", size: 24))
.fontWeight(.semibold)

// Full typography application
Text("Heading")
    .font(.system(size: 24, weight: .semibold))
    .foregroundColor(.white)

// With text decoration
Text("Underlined")
    .font(.system(size: 24, weight: .semibold))
    .underline()  // Always apply AFTER .font()
```

**SwiftUI modifier order:** Decoration (`.underline()`, `.strikethrough()`) must come AFTER `.font()`, never before.

### Kotlin (Jetpack Compose)

```kotlin
// Font family definition
val InterFontFamily = FontFamily(
    Font(R.font.inter_regular, FontWeight.Normal),
    Font(R.font.inter_medium, FontWeight.Medium),
    Font(R.font.inter_semibold, FontWeight.SemiBold),
    Font(R.font.inter_bold, FontWeight.Bold)
)

// Usage in Text composable
Text(
    text = "Heading",
    fontFamily = InterFontFamily,
    fontWeight = FontWeight.SemiBold,
    fontSize = 24.sp,
    lineHeight = 31.2.sp,
    letterSpacing = (-0.02).em
)

// Typography configuration
val Typography = Typography(
    bodyLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp
    ),
    titleLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 22.sp,
        lineHeight = 28.sp
    )
)
```

## Fallback Strategy Per Platform

### CSS / Tailwind

Use a font stack with system fallbacks:

```css
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

**Common fallback mapping:**

| Original Font | Fallback Stack |
|---------------|---------------|
| SF Pro | `Inter, -apple-system, system-ui` |
| SF Pro Display | `Inter, -apple-system` |
| SF Pro Text | `Inter, -apple-system` |
| Helvetica Neue | `Inter, Arial, sans-serif` |
| Roboto | `Inter, -apple-system, sans-serif` |
| Open Sans | `Inter, Source Sans Pro, sans-serif` |
| Montserrat | `Poppins, Inter, sans-serif` |
| Playfair Display | `Merriweather, Georgia, serif` |
| Custom/Unknown | `Inter, system-ui, sans-serif` |

### SwiftUI

When a custom font is unavailable, fall back to system font:

```swift
// Custom font available
.font(.custom("Inter", size: 24))
.fontWeight(.semibold)

// Fallback: system font with matching weight and design
.font(.system(size: 24, weight: .semibold, design: .default))
```

**System font design options:**

| Design | Use Case |
|--------|----------|
| `.default` | Standard sans-serif (most common fallback) |
| `.rounded` | Rounded variant (for playful designs) |
| `.serif` | Serif fallback |
| `.monospaced` | Code/technical content |

### Kotlin / Android

Fonts are bundled in `res/font/` directory with XML font family definition:

```xml
<?xml version="1.0" encoding="utf-8"?>
<font-family xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <font
        android:font="@font/inter_regular"
        android:fontStyle="normal"
        android:fontWeight="400"
        app:font="@font/inter_regular"
        app:fontStyle="normal"
        app:fontWeight="400" />
    <font
        android:font="@font/inter_semibold"
        android:fontStyle="normal"
        android:fontWeight="600"
        app:font="@font/inter_semibold"
        app:fontStyle="normal"
        app:fontWeight="600" />
</font-family>
```

If font is unavailable, Android falls back to the default system font (Roboto).

## Font Source Priority

When setting up fonts, search in this order:

1. **Google Fonts** (primary) -- free, widely available
2. **Font Squirrel** (fallback 1) -- free fonts with webfont kits
3. **Adobe Fonts** (fallback 2) -- requires subscription, report to user
4. **Manual** -- ask user to provide font files

## SF Symbols as Icon Font Fallback

When icon assets cannot be resolved, use SF Symbols as fallback on Apple platforms:

```swift
// SF Symbol usage
Image(systemName: "checkmark.circle.fill")
    .font(.system(size: 24))
    .foregroundColor(.green)
```

SF Symbols are only available on Apple platforms (iOS, macOS). For cross-platform projects, use platform-specific icon solutions.

## iOS Version Considerations

| iOS Version | Font Feature | Notes |
|-------------|-------------|-------|
| iOS 13+ | `.font(.custom())` | Basic custom font support |
| iOS 14+ | `@ScaledMetric` | Dynamic Type scaling for custom fonts |
| iOS 15+ | `.dynamicTypeSize()` | Fine-grained Dynamic Type control |
| iOS 16+ | Variable fonts | Full variable font axis support |

**Dynamic Type support checklist:**

- Use system font sizes when possible for automatic scaling
- Apply `.dynamicTypeSize()` modifier for custom size limits
- Test with Accessibility Inspector for font scaling behavior

## Text Auto-Resize Behavior

Figma's `textAutoResize` property affects how text frames behave:

| Figma Value | Behavior | SwiftUI | CSS |
|-------------|----------|---------|-----|
| `NONE` | Fixed frame, text may clip | `.frame(width:height:).clipped()` | `overflow: hidden` |
| `HEIGHT` | Width fixed, height adjusts | No `.lineLimit()` needed | `height: auto` |
| `WIDTH_AND_HEIGHT` | Both adjust | No constraints needed | `width: auto; height: auto` |
| `TRUNCATE` | Fixed frame, text truncated | `.lineLimit(N)` | `text-overflow: ellipsis` |

## Cross-References

- **Font setup:** Font Manager agent handles font downloading, installation, and platform configuration
- **Opacity on text:** `@references/opacity-extraction.md` -- text foreground color opacity
- **Color tokens:** `@references/color-extraction.md` -- text color extraction and application
- **Token mapping:** `@references/token-mapping.md` -- design token to code token mapping
