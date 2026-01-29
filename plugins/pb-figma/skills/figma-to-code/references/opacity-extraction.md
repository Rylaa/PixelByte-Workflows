# Opacity Extraction Reference

## Compound Opacity Calculation

Figma stores opacity at multiple levels. Calculate effective opacity:

```typescript
effectiveOpacity = fillOpacity * nodeOpacity
```

**Example:**
- Fill opacity: 0.8
- Node opacity: 0.5
- Effective: 0.8 * 0.5 = 0.4

## Extraction from Figma

```typescript
const nodeDetails = figma_get_node_details({
  file_key: "{file_key}",
  node_id: "{node_id}"
});

// Extract BOTH fill opacity and node opacity
const fillOpacity = nodeDetails.fills?.[0]?.opacity ?? 1.0;  // Default to 1.0 if undefined
const nodeOpacity = nodeDetails.opacity ?? 1.0;              // Default to 1.0 if undefined

// Calculate effective opacity (compound multiplication)
const effectiveOpacity = fillOpacity * nodeOpacity;
```

## Opacity Sources

| Source | Figma Property | Priority |
|--------|---------------|----------|
| Fill | `fills[0].opacity` | Primary |
| Node | `opacity` | Multiplier |
| Effect | `effects[].opacity` | Additive |

## Warning Conditions

Flag for review when:
- `effectiveOpacity < 0.1` - Nearly invisible
- `effectiveOpacity !== 1.0 AND effectiveOpacity !== 0.0` - Partial transparency
- Multiple opacity sources combined
- Border/stroke opacity < 0.8 - May appear faded
- Text opacity < 1.0 - May reduce readability

## Calculation Examples

| Fill Opacity | Node Opacity | Effective | Usage |
|--------------|--------------|-----------|-------|
| 0.4 | 1.0 | 0.4 | `.stroke(Color.white.opacity(0.4))` |
| 0.5 | 0.8 | 0.4 | `.opacity(0.4)` |
| undefined | 0.6 | 0.6 | `.opacity(0.6)` |
| 1.0 | 1.0 | 1.0 | No `.opacity()` modifier needed |

## SwiftUI Application

### Pattern 1: Color-level opacity (for fills, strokes, text)

```swift
// Border with opacity
RoundedRectangle(cornerRadius: 12)
    .stroke(Color.white.opacity(0.4), lineWidth: 1.0)

// Background with opacity
Rectangle()
    .fill(Color(hex: "#150200").opacity(0.8))

// Text with opacity
Text(title)
    .foregroundColor(Color(hex: "#333333").opacity(0.9))
```

### Pattern 2: View-level opacity (for gradients, overlays)

```swift
// Gradient overlay with view-level opacity
RadialGradient(
    gradient: Gradient(stops: [
        .init(color: Color(hex: "#f02912"), location: 0.0),
        .init(color: Color(hex: "#150200"), location: 1.0)
    ]),
    center: .center,
    startRadius: 0,
    endRadius: 200
)
.opacity(0.2)
```

## Critical Rules

1. **Always extract BOTH opacities** from `figma_get_node_details`
2. **Calculate effective opacity** before adding to spec
3. **opacity: 1.0** - Omit `.opacity()` modifier (SwiftUI default)
4. **opacity: 0.0** - Element is invisible, verify intentional
5. **Copy Usage column** from Design Tokens table directly into code

## Cross-Reference

See `@references/color-extraction.md` for color format details including hex extraction, RGBA conversion, and color token mapping.

## Platform-Specific Opacity Application

### CSS

```css
/* View-level opacity (affects entire element and children) */
.overlay {
  opacity: 0.5;
}

/* Color-level opacity via rgba alpha channel */
.card {
  background-color: rgba(21, 2, 0, 0.8);    /* #150200 at 80% */
  border: 1px solid rgba(255, 255, 255, 0.4); /* white at 40% */
  color: rgba(51, 51, 51, 0.9);               /* #333333 at 90% */
}

/* Color-level opacity via 8-character hex (RRGGBBAA) */
.element {
  background-color: #150200CC; /* #150200 at 80% (CC = 204/255 = 0.8) */
}
```

**When to use which:**
| Scenario | CSS Property | Example |
|----------|-------------|---------|
| Entire element transparent | `opacity` | `opacity: 0.5` |
| Background only | `rgba()` or 8-char hex | `background: rgba(0,0,0,0.5)` |
| Border only | `rgba()` on border-color | `border-color: rgba(255,255,255,0.4)` |
| Text only | `rgba()` on color | `color: rgba(0,0,0,0.7)` |

### Tailwind CSS

Tailwind uses the `/` opacity modifier syntax appended to color utilities:

```html
<!-- Background opacity -->
<div class="bg-black/50">         <!-- background-color: rgb(0 0 0 / 0.5) -->
<div class="bg-white/80">         <!-- background-color: rgb(255 255 255 / 0.8) -->
<div class="bg-[#150200]/80">     <!-- arbitrary color at 80% opacity -->

<!-- Text opacity -->
<p class="text-black/50">         <!-- color: rgb(0 0 0 / 0.5) -->
<p class="text-white/70">         <!-- color: rgb(255 255 255 / 0.7) -->

<!-- Border opacity -->
<div class="border border-white/40">  <!-- border-color at 40% opacity -->

<!-- Element-level opacity (affects entire element) -->
<div class="opacity-50">          <!-- opacity: 0.5 -->
<div class="opacity-80">          <!-- opacity: 0.8 -->
```

**Tailwind opacity modifier mapping:**

| Figma Opacity | Tailwind Modifier | Tailwind Class Example |
|---------------|-------------------|------------------------|
| 0.0 | `/0` | `bg-black/0` |
| 0.05 | `/5` | `bg-black/5` |
| 0.1 | `/10` | `bg-black/10` |
| 0.2 | `/20` | `bg-black/20` |
| 0.25 | `/25` | `bg-black/25` |
| 0.3 | `/30` | `bg-black/30` |
| 0.4 | `/40` | `bg-black/40` |
| 0.5 | `/50` | `bg-black/50` |
| 0.6 | `/60` | `bg-black/60` |
| 0.7 | `/70` | `bg-black/70` |
| 0.75 | `/75` | `bg-black/75` |
| 0.8 | `/80` | `bg-black/80` |
| 0.9 | `/90` | `bg-black/90` |
| 0.95 | `/95` | `bg-black/95` |
| 1.0 | `/100` or omit | `bg-black` |

For non-standard opacity values, use arbitrary values: `bg-black/[0.35]`.

### SwiftUI

```swift
// Color-level opacity (for fills, strokes, text)
Color.white.opacity(0.4)
Color(hex: "#150200").opacity(0.8)

// View-level opacity (for gradients, overlays, entire views)
someView.opacity(0.2)

// 8-character hex with ARGB format
// Note: SwiftUI uses ARGB order (not RGBA)
Color(hex: "#80FFFFFF")  // white at ~50% opacity (0x80 = 128/255)
```

**Key rules:**
1. **Primary source: Usage column** from Design Tokens table - copy exactly as shown
2. **Never ignore opacity modifiers** - if Usage shows `.opacity(X)`, include it
3. **opacity: 1.0** - no `.opacity()` modifier needed (SwiftUI default)
4. **opacity: 0.0** - element is invisible, verify intentional

## Multiple Fills with Different Opacities

Figma nodes can have multiple fills, each with its own opacity. Extract and apply each fill individually.

### Extraction

```typescript
const fills = nodeDetails.fills ?? [];
const nodeOpacity = nodeDetails.opacity ?? 1.0;

const processedFills = fills
  .filter(fill => fill.visible !== false)
  .map((fill, index) => ({
    index,
    type: fill.type,
    color: fill.color,
    fillOpacity: fill.opacity ?? 1.0,
    effectiveOpacity: (fill.opacity ?? 1.0) * nodeOpacity
  }));
```

### Platform Examples

**CSS / Tailwind - multiple backgrounds:**
```css
.multi-fill {
  /* Layered backgrounds - last listed is bottom layer */
  background:
    linear-gradient(rgba(0, 0, 0, 0.3), rgba(0, 0, 0, 0.3)),  /* overlay at 30% */
    rgba(21, 2, 0, 0.8);                                        /* base at 80% */
}
```

**SwiftUI - stacked fills:**
```swift
ZStack {
  Color(hex: "#150200").opacity(0.8)  // base fill
  Color.black.opacity(0.3)            // overlay fill
}
```

## Compliance Tolerance for Opacity Values

When validating opacity compliance between the design spec and generated code:

| Property | Tolerance | Rule |
|----------|-----------|------|
| Opacity value | Exact match | Opacity values must match the spec exactly (no rounding tolerance) |
| Color hex | Exact match | Hex values must be identical |
| Effective opacity | Exact calculation | `fillOpacity * nodeOpacity` must be computed precisely |

**Rounding guidance:**
- Preserve Figma's original precision (typically 2 decimal places)
- Do not round `0.35` to `0.4` or `0.85` to `0.9`
- For Tailwind, if the exact value does not map to a built-in modifier, use arbitrary syntax: `/[0.35]`
- For CSS, use the exact decimal value from the spec
