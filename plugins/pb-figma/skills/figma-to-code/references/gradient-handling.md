# Gradient Handling Reference

> **Used by:** design-analyst, design-validator, code-generator-*, compliance-checker

> Shared gradient domain knowledge for the Figma-to-code pipeline.
> Referenced by: design-analyst, code-generator-react, code-generator-swiftui

---

## Figma Gradient Types

Figma exposes four gradient types via the `fills` array on nodes:

| Figma API Type | Short Name | Description |
|----------------|-----------|-------------|
| `GRADIENT_LINEAR` | `LINEAR` | Linear gradient with angle/direction |
| `GRADIENT_RADIAL` | `RADIAL` | Radial gradient from center outward |
| `GRADIENT_ANGULAR` | `ANGULAR` | Conic/angular gradient (rainbow/sweep effect) |
| `GRADIENT_DIAMOND` | `DIAMOND` | Diamond-shaped gradient |

---

## Gradient Extraction Rules

### Detection

- **Always use `figma_get_node_details`** (NOT `figma_get_design_tokens`) to get the `fills` array
- Check `fills[].type` for gradient types: `GRADIENT_LINEAR`, `GRADIENT_RADIAL`, `GRADIENT_ANGULAR`, `GRADIENT_DIAMOND`

### Extraction Query Pattern

```typescript
const nodeDetails = figma_get_node_details({
  file_key: "{file_key}",
  node_id: "{node_id}"
});

// Check if node has gradient fills
const gradientFill = nodeDetails.fills?.find(fill =>
  fill.type?.includes('GRADIENT')
);

if (gradientFill) {
  // Map Figma gradient type to output format
  const gradientTypeMap = {
    'GRADIENT_LINEAR': 'LINEAR',
    'GRADIENT_RADIAL': 'RADIAL',
    'GRADIENT_ANGULAR': 'ANGULAR',
    'GRADIENT_DIAMOND': 'DIAMOND'
  };

  const gradientType = gradientTypeMap[gradientFill.type] || gradientFill.type;

  // Extract ALL gradient stops with EXACT positions (4 decimals)
  const stops = gradientFill.gradientStops.map(stop => {
    // Convert RGB (0-1 floats) to hex
    const r = Math.round(stop.color.r * 255);
    const g = Math.round(stop.color.g * 255);
    const b = Math.round(stop.color.b * 255);
    const hex = `#${r.toString(16).padStart(2, '0')}${g.toString(16).padStart(2, '0')}${b.toString(16).padStart(2, '0')}`;

    // Extract opacity (default to 1.0 if not present)
    const opacity = stop.color.a ?? 1.0;

    // Round position to 4 decimal places (NOT 2!)
    const position = Math.round(stop.position * 10000) / 10000;

    return { position, hex, opacity };
  });
}
```

### Core Rules

- Extract **ALL** gradient stops from `gradientStops` array (no truncation)
- **Preserve EXACT position values** - Round to 4 decimal places (0.1673, NOT 0.17)
- Convert RGB colors (0-1 floats) to hex format (#bc82f3)
- Extract opacity from `stop.color.a` (default to 1.0 if missing)
- Include opacity for EVERY stop: `0.1673: #bc82f3 (opacity: 1.0)`

---

## 4-Decimal Precision

Gradient stop locations MUST use exactly 4 decimal places:

- `0.1673` -- correct
- `0.17` -- WRONG (rounded)
- `0.697` -- WRONG (missing trailing zero)
- `0.6970` -- correct (trailing zero preserved)

---

## Gradient Stop Format

Standard format for Implementation Spec:

```
{ color: #hex, location: 0.xxxx, opacity: n.n }
```

Example in Implementation Spec output:

```markdown
### Text with Gradient

**Component:** HeadingText
- **Gradient Type:** ANGULAR
- **Stops:**
  - 0.1673: #bc82f3 (opacity: 1.0)
  - 0.2365: #f4b9ea (opacity: 1.0)
  - 0.3518: #8d98ff (opacity: 1.0)
  - 0.5815: #aa6eee (opacity: 1.0)
  - 0.697: #ff6777 (opacity: 1.0)
  - 0.8095: #ffba71 (opacity: 1.0)
  - 0.9241: #c686ff (opacity: 1.0)
```

---

## Platform-Specific Mapping

### CSS / Tailwind

**Gradient type mapping:**

| Figma Type | CSS Function | Example |
|------------|--------------|---------|
| LINEAR | `linear-gradient()` | `linear-gradient(90deg, #F00, #00F)` |
| RADIAL | `radial-gradient()` | `radial-gradient(circle, #F00, #00F)` |
| ANGULAR | `conic-gradient()` | `conic-gradient(from 0deg, #F00, #00F)` |
| DIAMOND | `conic-gradient()` | Treat as conic |

**Angle conversion (Figma to CSS):**

| Figma Angle | CSS Direction | Tailwind |
|-------------|---------------|----------|
| 0deg | `to right` | `bg-gradient-to-r` |
| 90deg | `to bottom` | `bg-gradient-to-b` |
| 180deg | `to left` | `bg-gradient-to-l` |
| 270deg | `to top` | `bg-gradient-to-t` |
| 45deg | `to bottom right` | `bg-gradient-to-br` |

**LINEAR gradient examples:**

```tsx
// Tailwind arbitrary value
<div className="bg-[linear-gradient(180deg,#FF0000_0%,#00FF00_50%,#0000FF_100%)]">

// Tailwind gradient utilities (limited colors)
<div className="bg-gradient-to-b from-red-500 via-green-500 to-blue-500">

// CSS-in-JS / style prop
<div style={{
  background: 'linear-gradient(180deg, #FF0000 0%, #00FF00 50%, #0000FF 100%)'
}}>
```

**RADIAL gradient examples:**

```tsx
// CSS arbitrary value
<div className="bg-[radial-gradient(circle,#FFFF00_0%,#FF00FF_100%)]">

// Style prop
<div style={{
  background: 'radial-gradient(circle at center, #FFFF00 0%, #FF00FF 100%)'
}}>
```

**ANGULAR (conic) gradient examples:**

```tsx
// Conic gradient (for angular/sweep gradients)
<div className="bg-[conic-gradient(from_0deg,#FF0000,#00FF00,#0000FF,#FF0000)]">

// With specific stop positions
<div style={{
  background: `conic-gradient(
    from 0deg at center,
    #bc82f3 16.73%,
    #f4b9ea 23.65%,
    #8d98ff 35.18%,
    #aa6eee 58.15%,
    #ff6777 69.70%,
    #ffba71 80.95%,
    #c686ff 92.41%
  )`
}}>
```

**CSS/Tailwind critical rules:**

1. **Preserve ALL gradient stops** - Every stop from spec must appear
2. **Use exact percentages** - Convert decimal to percent (0.1673 -> 16.73%)
3. **Preserve precision** - Don't round 16.73% to 17%
4. **Use `bg-clip-text text-transparent`** for text gradients
5. **Prefer Tailwind utilities** when gradient is simple (2-3 colors, standard angles)
6. **Use style prop** for complex gradients (4+ colors, precise positions)

### SwiftUI

**Gradient type mapping:**

| Figma Type | SwiftUI Type | Example |
|------------|--------------|---------|
| LINEAR | `LinearGradient` | `LinearGradient(stops: [...], startPoint: .top, endPoint: .bottom)` |
| RADIAL | `RadialGradient` | `RadialGradient(stops: [...], center: .center, startRadius: 0, endRadius: 100)` |
| ANGULAR | `AngularGradient` | `AngularGradient(stops: [...], center: .center)` |
| DIAMOND | `AngularGradient` | `AngularGradient(stops: [...], center: .center)` (treat as angular) |

**Angle to startPoint/endPoint conversion (LINEAR only):**

| Angle | startPoint | endPoint | Direction |
|-------|------------|----------|-----------|
| 0deg | `.leading` | `.trailing` | Left to right |
| 90deg | `.top` | `.bottom` | Top to bottom |
| 180deg | `.trailing` | `.leading` | Right to left |
| 270deg | `.bottom` | `.top` | Bottom to top |
| 45deg | `.topLeading` | `.bottomTrailing` | Diagonal |
| 135deg | `.topTrailing` | `.bottomLeading` | Diagonal |

**ANGULAR gradient example:**

```swift
Text("Type a scene to generate your video")
    .font(.system(size: 14, weight: .regular))
    .foregroundStyle(
        AngularGradient(
            stops: [
                Gradient.Stop(color: Color(hex: "#bc82f3"), location: 0.1673),
                Gradient.Stop(color: Color(hex: "#f4b9ea"), location: 0.2365),
                Gradient.Stop(color: Color(hex: "#8d98ff"), location: 0.3518),
                Gradient.Stop(color: Color(hex: "#aa6eee"), location: 0.5815),
                Gradient.Stop(color: Color(hex: "#ff6777"), location: 0.6970),
                Gradient.Stop(color: Color(hex: "#ffba71"), location: 0.8095),
                Gradient.Stop(color: Color(hex: "#c686ff"), location: 0.9241)
            ],
            center: .center
        )
    )
```

**LINEAR gradient example:**

```swift
Text("Gradient text")
    .foregroundStyle(
        LinearGradient(
            stops: [
                Gradient.Stop(color: Color(hex: "#FF0000"), location: 0.0),
                Gradient.Stop(color: Color(hex: "#0000FF"), location: 1.0)
            ],
            startPoint: .leading,
            endPoint: .trailing
        )
    )
```

**RADIAL gradient example:**

```swift
Text("Radial gradient")
    .foregroundStyle(
        RadialGradient(
            stops: [
                Gradient.Stop(color: Color(hex: "#FFFF00"), location: 0.0),
                Gradient.Stop(color: Color(hex: "#FF00FF"), location: 1.0)
            ],
            center: .center,
            startRadius: 0,
            endRadius: 100
        )
    )
```

**SwiftUI critical rules:**

1. **Preserve ALL gradient stops** - Every stop from Implementation Spec must appear in output
2. **4-decimal precision** - Use exactly 4 decimal places (0.1673, not 0.17 or 0.167)
3. **Keep trailing zeros** - 0.6970 NOT 0.697 (maintains precision from Figma)
4. **Never round** - Use exact location values from spec
5. **Use `Color(hex:)` for stop colors** - requires Color+Hex extension
6. **Minimum iOS 15** - `.foregroundStyle()` requires iOS 15+, add `@available(iOS 15.0, *)` if needed

---

## Gradient Text Handling

### CSS / Tailwind

Use `background-clip: text` with transparent text fill:

```tsx
// Text gradient using background-clip
<span
  className="bg-clip-text text-transparent bg-[linear-gradient(90deg,#FF0000,#0000FF)]"
>
  Gradient Text
</span>

// Conic gradient text
<span
  className="bg-clip-text text-transparent"
  style={{
    background: `conic-gradient(
      from 0deg,
      #bc82f3 16.73%,
      #f4b9ea 23.65%,
      #8d98ff 35.18%,
      #c686ff 92.41%
    )`
  }}
>
  Rainbow Text
</span>
```

### SwiftUI

Use `.foregroundStyle()` (NOT `.foregroundColor()` which does not support gradients):

```swift
Text("Gradient text")
    .foregroundStyle(
        AngularGradient(
            stops: [ /* ... */ ],
            center: .center
        )
    )
```

**Common mistake:**

```swift
// WRONG - Won't compile
.foregroundColor(LinearGradient(...))

// CORRECT
.foregroundStyle(LinearGradient(...))
```

---

## Common Issues

### Opacity in Gradient Stops

- Extract per-stop opacity from `stop.color.a` (defaults to 1.0)
- In SwiftUI: Use hex colors with alpha channel for per-stop opacity (e.g., `#80FF0000`)
- **Do NOT** use `.opacity()` modifier with gradients -- it applies uniformly to the entire gradient

### Direction Mapping

- Figma angles must be converted to platform-specific directions
- CSS uses degrees directly in `linear-gradient(Ndeg, ...)`
- SwiftUI LINEAR uses `.startPoint` / `.endPoint` (see conversion table above)

### Angular Start Angle

- Figma ANGULAR gradients start at 0deg
- CSS `conic-gradient()` uses `from Ndeg` syntax
- SwiftUI `AngularGradient` defaults to center; angle offset via `startAngle`

### Precision Loss

- Hardcoding only 2-3 stops loses gradient fidelity
- Rounding positions to 0.0, 0.5, 1.0 shifts gradient balance
- Always preserve exact decimal values from Figma/spec

### Performance

- Complex gradients with 5+ stops may impact rendering performance
- Add warnings for gradients with many stops

---

## Platform Requirements

- **CSS/Tailwind gradient text:** Requires `background-clip: text` (widely supported)
- **SwiftUI gradient text:** Requires iOS 15.0+ / macOS 12.0+ for `.foregroundStyle()` with gradients
- **SwiftUI `AngularGradient` on Text:** Requires iOS 15.0+

### Compliance Checklist

- Add iOS 15+ platform requirement to Compliance section when gradient text is detected
- Warn if gradient has 5+ stops (performance impact)
- Map Figma gradient type to platform equivalent in spec output
