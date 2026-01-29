# Color Extraction Reference

## Figma Color Format

Figma represents colors as normalized RGBA values in the range `0.0` to `1.0`:

```typescript
interface FigmaColor {
  r: number;  // 0.0 - 1.0
  g: number;  // 0.0 - 1.0
  b: number;  // 0.0 - 1.0
  a: number;  // 0.0 - 1.0 (fill-level opacity)
}
```

## RGB to Hex Conversion

Convert Figma's `0-1` range to standard hex:

```typescript
function rgbToHex(r: number, g: number, b: number): string {
  const toHex = (v: number) => Math.round(v * 255).toString(16).padStart(2, '0');
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`.toUpperCase();
}

// Example: { r: 0.95, g: 0.95, b: 0.05 } --> "#F2F20D"
```

## Opacity: Fill vs Node

Figma stores opacity at two levels that must be multiplied:

| Source | Figma Property | Default | Description |
|--------|---------------|---------|-------------|
| Fill opacity | `fills[0].opacity` | 1.0 | Opacity of the specific fill |
| Node opacity | `opacity` | 1.0 | Opacity of the entire node |
| Effective | `fillOpacity * nodeOpacity` | -- | What the user sees |

```typescript
// Extract from figma_get_node_details
const fillOpacity = nodeDetails.fills?.[0]?.opacity ?? 1.0;
const nodeOpacity = nodeDetails.opacity ?? 1.0;
const effectiveOpacity = fillOpacity * nodeOpacity;
```

**Cross-reference:** `@references/opacity-extraction.md` for full compound opacity calculation, warning conditions, and platform-specific application.

## Color Table Format (Design Spec)

All component specs include a color table with mandatory fill opacity:

```markdown
| Property | Color | Opacity | Usage |
|----------|-------|---------|-------|
| Background | #000000 | 1.0 | `.background(Color(hex: "#000000"))` |
| Card Background | #f2f20d | 0.05 | `.background(Color(hex: "#f2f20d").opacity(0.05))` |
| Text Primary | #ffffff | 1.0 | `.foregroundColor(.white)` |
| Text Secondary | #ffffff | 0.7 | `.foregroundColor(.white.opacity(0.7))` |
| Border | #414141 | 1.0 | `.stroke(Color(hex: "#414141"))` |
```

## Color Formats Per Platform

### CSS

```css
/* Hex format (opacity = 1.0) */
color: #FFFFFF;
background-color: #150200;

/* RGBA format (opacity < 1.0) */
color: rgba(255, 255, 255, 0.7);
background-color: rgba(242, 242, 13, 0.05);

/* CSS custom properties */
:root {
  --color-primary: #F2F20D;
  --color-text: #FFFFFF;
  --color-text-secondary: rgba(255, 255, 255, 0.7);
}

/* Usage */
color: var(--color-text);
background-color: var(--color-primary);
```

### Tailwind CSS

```tsx
// Hex colors (opacity = 1.0)
<div className="bg-[#150200] text-white">

// Hex colors with opacity modifier
<div className="bg-[#F2F20D]/5">      // 5% opacity
<p className="text-white/70">          // 70% opacity text
<div className="border-white/40">      // 40% opacity border

// CSS variable with opacity modifier
<div className="bg-[var(--color-primary)]/50">

// Named Tailwind colors (when configured)
<div className="bg-primary text-foreground">
```

**Tailwind opacity pattern decision table:**

| Spec Opacity | Tailwind Pattern | Example |
|-------------|------------------|---------|
| 1.0 | No modifier | `text-white` or `text-[#FFFFFF]` |
| 0.8 | `/80` suffix | `text-white/80` |
| 0.5 | `/50` suffix | `bg-[#FF0000]/50` |
| 0.05 | `/5` suffix | `bg-[#F2F20D]/5` |
| 0.0 | Skip (invisible) | Verify intentional |

**CSS fallback (when Tailwind modifiers are insufficient):**

```tsx
// Using rgba()
<div style={{ backgroundColor: 'rgba(255, 255, 255, 0.4)' }}>

// Using CSS custom property with separate opacity
<div style={{ backgroundColor: 'var(--color-primary)', opacity: 0.5 }}>
```

### SwiftUI

```swift
// Opacity = 1.0: use shorthand for common colors
.foregroundColor(.white)   // #FFFFFF
.foregroundColor(.black)   // #000000

// Opacity = 1.0: hex colors
.background(Color(hex: "#150200"))

// Opacity < 1.0: use .opacity() modifier
.foregroundColor(.white.opacity(0.7))
.background(Color(hex: "#f2f20d").opacity(0.05))
.foregroundColor(Color(hex: "#CCCCCC").opacity(0.6))

// Border with opacity
.overlay(
    RoundedRectangle(cornerRadius: 12)
        .stroke(Color.white.opacity(0.4), lineWidth: 1)
)
```

**SwiftUI color decision table:**

| Hex Value | Opacity | SwiftUI Code |
|-----------|---------|--------------|
| #FFFFFF | 1.0 | `.white` |
| #000000 | 1.0 | `.black` |
| #FFFFFF | 0.7 | `.white.opacity(0.7)` |
| #F2F20D | 1.0 | `Color(hex: "#F2F20D")` |
| #F2F20D | 0.05 | `Color(hex: "#F2F20D").opacity(0.05)` |

### Hex-Alpha (ARGB) Format for SwiftUI

The `Color+Hex` extension supports 8-character ARGB hex strings (alpha first):

```
#40FFFFFF --> Alpha: 0x40 = 64/255 = 0.25, Color: #FFFFFF (white)
#80FF0000 --> Alpha: 0x80 = 128/255 = 0.50, Color: #FF0000 (red)
#FF00FF00 --> Alpha: 0xFF = 1.0,            Color: #00FF00 (green)
```

Prefer the `.opacity()` modifier for readability:

```swift
// Preferred: explicit opacity modifier
Color(hex: "#FFFFFF").opacity(0.25)

// Alternative: 8-char ARGB format
Color(hex: "#40FFFFFF")
```

## Color Token Mapping

| Figma Style Name | CSS Variable | Tailwind | SwiftUI |
|------------------|-------------|----------|---------|
| Primary | `var(--color-primary)` | `text-primary` / `bg-primary` | `Color.primary` or `Color(hex:)` |
| Background | `var(--color-bg)` | `bg-background` | `.background(Color(hex:))` |
| Text/Primary | `var(--color-text)` | `text-foreground` | `.foregroundColor(.white)` |
| Text/Secondary | `var(--color-text-secondary)` | `text-muted-foreground` | `.foregroundColor(.white.opacity(0.7))` |
| Border | `var(--color-border)` | `border-border` | `.stroke(Color(hex:))` |

## Special Cases

### Transparent Colors (a < 0.01)

Skip fills with near-zero alpha -- they are invisible and should not generate code:

```typescript
if (fill.opacity < 0.01 || fill.color.a < 0.01) {
  // Skip this fill -- effectively transparent
  continue;
}
```

### Semi-Transparent Fills

When fill opacity is between 0.01 and 0.99, apply platform opacity modifier:

| Platform | Pattern |
|----------|---------|
| Tailwind | `bg-[#HEX]/XX` where XX = opacity * 100 |
| CSS | `rgba(r, g, b, opacity)` |
| SwiftUI | `Color(hex: "#HEX").opacity(X.X)` |

### Multiple Fills (Layered)

Figma nodes can have multiple fills rendered bottom-to-top. Handle as layered backgrounds:

```tsx
// Tailwind/CSS: use multiple background layers or stacked divs
<div className="relative">
  <div className="absolute inset-0 bg-[#000000]" />          {/* Bottom fill */}
  <div className="absolute inset-0 bg-[#F2F20D]/5" />        {/* Top fill */}
  <div className="relative z-10">{children}</div>
</div>
```

```swift
// SwiftUI: use ZStack or layered backgrounds
ZStack {
    Color(hex: "#000000")                          // Bottom fill
    Color(hex: "#F2F20D").opacity(0.05)            // Top fill
    content
}
```

### Inline Text Color Variations

When text contains segments with different colors, use concatenation:

```swift
// SwiftUI: Text concatenation with + operator
Text("Let's fix your ")
    .font(.system(size: 24, weight: .semibold))
    .foregroundColor(.white)
+
Text("Hook")
    .font(.system(size: 24, weight: .semibold))
    .foregroundColor(Color(hex: "#F2F20D"))
    .underline()
```

## Color+Hex Swift Extension

Include this extension whenever generated SwiftUI code uses `Color(hex:)`:

```swift
extension Color {
  init(hex: String) {
    let hex = hex.trimmingCharacters(in: CharacterSet.alphanumerics.inverted)
    var int: UInt64 = 0
    Scanner(string: hex).scanHexInt64(&int)
    let a, r, g, b: UInt64
    switch hex.count {
    case 3: // RGB (12-bit)
      (a, r, g, b) = (255, (int >> 8) * 17, (int >> 4 & 0xF) * 17, (int & 0xF) * 17)
    case 6: // RGB (24-bit)
      (a, r, g, b) = (255, int >> 16, int >> 8 & 0xFF, int & 0xFF)
    case 8: // ARGB (32-bit)
      (a, r, g, b) = (int >> 24, int >> 16 & 0xFF, int >> 8 & 0xFF, int & 0xFF)
    default:
      (a, r, g, b) = (255, 0, 0, 0)
    }
    self.init(
      .sRGB,
      red: Double(r) / 255,
      green: Double(g) / 255,
      blue: Double(b) / 255,
      opacity: Double(a) / 255
    )
  }
}
```

**Supported formats:**

| Format | Length | Example | Parsing |
|--------|--------|---------|---------|
| RGB (12-bit) | 3 chars | `#F0D` | Each nibble expanded (F->FF, 0->00, D->DD) |
| RGB (24-bit) | 6 chars | `#F2F20D` | Standard hex |
| ARGB (32-bit) | 8 chars | `#40FFFFFF` | Alpha first, then RGB |

## Compliance Verification

Compliance checker validates colors with these grep patterns:

```bash
# SwiftUI color checks
Grep("Color(hex:", path="{component_file_path}")
Grep("Color(\"", path="{component_file_path}")
Grep(".foregroundColor(", path="{component_file_path}")
Grep(".background(", path="{component_file_path}")
Grep(".opacity([0-9]", path="{component_file_path}")
```

## Cross-References

- **Opacity details:** `@references/opacity-extraction.md` -- compound opacity calculation and platform application
- **Gradient colors:** `@references/gradient-handling.md` -- gradient fill extraction and code generation
- **Token mapping:** `@references/token-mapping.md` -- design token to code mapping
