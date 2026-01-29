# Shadow and Blur Effects Reference

> **Used by:** design-analyst, design-validator, code-generator-*, compliance-checker

Comprehensive reference for shadow, blur, and glass effects across Figma-to-code generation pipelines.

## Figma Effect Types

### Shadow Effects

Figma exposes two shadow types on any node's `effects` array:

| Type | Description | Properties |
|------|-------------|------------|
| `DROP_SHADOW` | Outer shadow cast behind the element | color (rgba), offset (x, y), radius, spread |
| `INNER_SHADOW` | Inward shadow rendered inside the element boundary | color (rgba), offset (x, y), radius, spread |

**Property breakdown:**

| Property | Type | Notes |
|----------|------|-------|
| `color` | `{ r, g, b, a }` | Float 0-1 per channel; convert to rgba() or hex+opacity |
| `offset.x` | number (px) | Horizontal offset; positive = right |
| `offset.y` | number (px) | Vertical offset; positive = down |
| `radius` | number (px) | Blur radius (Gaussian blur size) |
| `spread` | number (px) | Expansion/contraction of shadow shape |

### Blur Effects

| Type | Description | Properties |
|------|-------------|------------|
| `LAYER_BLUR` | Blurs the element itself | radius |
| `BACKGROUND_BLUR` | Blurs content behind the element (frosted glass) | radius |

---

## Glass Effect Detection

### Detection Pattern (design-analyst)

When a component has a low-opacity fill combined with a corner radius, it represents a glass/frosted-glass appearance:

```
IF fill.opacity <= 0.3 AND fill.opacity > 0 AND cornerRadius > 0:
  -> Mark as "Glass Effect: true"
  -> Add: | **Glass Effect** | true -- low-opacity fill suggests frosted glass appearance |
  -> Add: | **Glass Tint** | {fill.color} at {fill.opacity} opacity |
```

### Glass Tint

The Glass Tint is the overlay color extracted from the low-opacity fill:

| Spec Property | Example Value | Meaning |
|---------------|---------------|---------|
| **Glass Effect** | `true` | Element uses frosted glass appearance |
| **Glass Tint** | `#ffae96 at 0.10 opacity` | Overlay color and its original opacity |

### Example Spec Output

```markdown
### SaveButton

| Property | Value |
|----------|-------|
| **Element** | Button |
| **Dimensions** | `width: 311, height: 48` |
| **Corner Radius** | `32px` |
| **Glass Effect** | true -- low-opacity fill suggests frosted glass appearance |
| **Glass Tint** | #ffae96 at 0.10 opacity |
| **Children** | ButtonLabel |
```

---

## Platform Mapping

### CSS

| Figma Effect | CSS Property | Example |
|--------------|-------------|---------|
| DROP_SHADOW | `box-shadow` | `box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1)` |
| DROP_SHADOW (on img/svg) | `filter: drop-shadow()` | `filter: drop-shadow(0px 4px 6px rgba(0, 0, 0, 0.1))` |
| INNER_SHADOW | `box-shadow: inset` | `box-shadow: inset 0px 2px 4px rgba(0, 0, 0, 0.06)` |
| LAYER_BLUR | `filter: blur()` | `filter: blur(4px)` |
| BACKGROUND_BLUR | `backdrop-filter: blur()` | `backdrop-filter: blur(16px)` |

### Tailwind CSS

| Figma Effect | Tailwind Utility | Arbitrary Value |
|--------------|-----------------|-----------------|
| DROP_SHADOW `0 1px 2px rgba(0,0,0,0.05)` | `shadow-sm` | -- |
| DROP_SHADOW `0 4px 6px rgba(0,0,0,0.1)` | `shadow-md` | -- |
| DROP_SHADOW (custom) | -- | `shadow-[0_4px_6px_rgba(0,0,0,0.1)]` |
| DROP_SHADOW (on img/svg) | `drop-shadow-md` | `drop-shadow-[0_4px_6px_rgba(0,0,0,0.1)]` |
| INNER_SHADOW | `shadow-inner` | `shadow-[inset_0_2px_4px_rgba(0,0,0,0.06)]` |
| BACKGROUND_BLUR | `backdrop-blur-md` | `backdrop-blur-[16px]` |
| LAYER_BLUR | `blur-sm` | `blur-[4px]` |

**Standard shadow mapping from design-analyst:**

```
0 1px 2px rgba(0,0,0,0.05) -> shadow-sm
0 4px 6px rgba(0,0,0,0.1)  -> shadow-md
Custom                      -> shadow-[0_4px_6px_rgba(0,0,0,0.1)]
```

### SwiftUI

| Figma Effect | SwiftUI Modifier | Example |
|--------------|-----------------|---------|
| DROP_SHADOW | `.shadow()` | `.shadow(color: Color.black.opacity(0.1), radius: 8, x: 0, y: 4)` |
| INNER_SHADOW | `.shadow()` inside `.overlay()` | Use inverted mask technique |
| LAYER_BLUR | `.blur()` | `.blur(radius: 4)` |
| BACKGROUND_BLUR | `.ultraThinMaterial` / `.glassEffect()` | See Glass Effect section below |

---

## SwiftUI Glass Effect (iOS 26+ Liquid Glass)

When a component has `Glass Effect: true` in the spec, generate iOS 26 Liquid Glass code with backward-compatible fallback.

### Button Pattern

```swift
// For buttons with Glass Effect: true
if #available(iOS 26.0, *) {
    Button(action: { /* action */ }) {
        Text("Save with Pro")
            .font(.system(size: 16, weight: .semibold))
            .foregroundStyle(.white)
    }
    .buttonStyle(.glassProminent)
    .tint(Color(hex: "#ffae96"))  // Glass Tint color from spec
    .frame(width: 311, height: 48)
    .clipShape(Capsule())
} else {
    // Fallback for iOS < 26
    Button(action: { /* action */ }) {
        Text("Save with Pro")
            .font(.system(size: 16, weight: .semibold))
            .foregroundStyle(.white)
    }
    .frame(width: 311, height: 48)
    .background(
        .ultraThinMaterial,
        in: Capsule()
    )
    .overlay(
        Capsule()
            .fill(Color(hex: "#ffae96").opacity(0.10))
    )
}
```

### Container Pattern

```swift
// For non-button containers with Glass Effect: true
if #available(iOS 26.0, *) {
    content
        .glassEffect(.regular)
        .tint(Color(hex: "{glass_tint_color}"))
} else {
    content
        .background(.ultraThinMaterial)
        .overlay(
            RoundedRectangle(cornerRadius: {radius})
                .fill(Color(hex: "{glass_tint_color}").opacity({glass_tint_opacity}))
        )
        .clipShape(RoundedRectangle(cornerRadius: {radius}))
}
```

### Glass Effect Rules

1. Always use `#available(iOS 26.0, *)` check -- never use `@available` at struct level for this
2. For buttons: use `.buttonStyle(.glassProminent)` with `.tint()` for the glass tint color
3. For containers: use `.glassEffect(.regular)` with `.tint()`
4. Fallback uses `.ultraThinMaterial` as background + overlay with tint color at original opacity
5. If corner radius >= height/2, use `Capsule()` instead of `RoundedRectangle`
6. Glass Tint color comes from the spec's `Glass Tint` property (produced by design-analyst when `fill.opacity <= 0.3 AND cornerRadius > 0`)

---

## SwiftUI Modifier Ordering

Shadow placement in the modifier chain is critical. The correct order is:

```swift
.padding()           // 1. Internal padding (affects content)
.frame()             // 2. Size constraints
.background()        // 3. Background color/gradient
.clipShape()         // 4. Clip to shape (BEFORE overlay)
.overlay()           // 5. Border stroke (AFTER clipShape)
.shadow()            // 6. Shadow (outermost)
```

### Why Order Matters

| Wrong Order | Problem |
|-------------|---------|
| `.overlay()` before `.clipShape()` | Border gets clipped, corners cut off |
| `.background()` after `.clipShape()` | Background bleeds outside rounded corners |
| `.shadow()` before `.clipShape()` | Shadow shape doesn't match clipped shape |
| `.frame()` after `.clipShape()` | Frame size may not match clipped content |

### Example -- Correct Modifier Chain

```swift
VStack { /* content */ }
    .padding(24)
    .background(Color("CardBackground"))
    .cornerRadius(16)
    .shadow(color: Color.black.opacity(0.1), radius: 8, x: 0, y: 4)
```

---

## Compliance Tolerances

The compliance-checker uses these tolerances when validating shadow and effect values:

| Aspect | Tolerance | Validation Check |
|--------|-----------|------------------|
| Shadows/Effects | Visual match | Shadow offset, blur, spread, color match |
| Glass Effect | Visual match | Glass/translucent elements render with proper material effect |
| Colors | Exact hex match | Background, text, border, shadow colors identical |
| Corner Radius | Exact match | All corners match spec values |
| Spacing | +/-4px | Padding, margin, gap values match design |
| Dimensions | +/-2px | Width, height match frame properties |

### Compliance Checklist Items

- [ ] **Shadows/Effects** -- Shadow and blur effects applied
- [ ] **Glass Effect** -- Glass/translucent elements render with proper material effect

---

## Decision Table: Effect Type Selection

| Figma Effect | Element Type | Platform | Generated Code |
|--------------|-------------|----------|---------------|
| DROP_SHADOW | Block element | CSS | `box-shadow: {x}px {y}px {radius}px {spread}px {rgba}` |
| DROP_SHADOW | Image/SVG | CSS | `filter: drop-shadow({x}px {y}px {radius}px {rgba})` |
| DROP_SHADOW | Any | Tailwind | `shadow-{size}` or `shadow-[...]` |
| DROP_SHADOW | Any | SwiftUI | `.shadow(color:radius:x:y:)` |
| INNER_SHADOW | Block element | CSS | `box-shadow: inset {x}px {y}px {radius}px {spread}px {rgba}` |
| BACKGROUND_BLUR + low fill | Container | CSS | `backdrop-filter: blur({r}px)` + semi-transparent bg |
| BACKGROUND_BLUR + low fill | Container | Tailwind | `backdrop-blur-{size}` + `bg-{color}/{opacity}` |
| BACKGROUND_BLUR + low fill | Button | SwiftUI | `.buttonStyle(.glassProminent)` + `.tint()` (iOS 26+) |
| BACKGROUND_BLUR + low fill | Container | SwiftUI | `.glassEffect(.regular)` + `.tint()` (iOS 26+) |
| LAYER_BLUR | Any | CSS | `filter: blur({r}px)` |
| LAYER_BLUR | Any | SwiftUI | `.blur(radius: {r})` |
