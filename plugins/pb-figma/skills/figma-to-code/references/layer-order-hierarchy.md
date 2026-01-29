# Layer Order & Hierarchy Reference

## Core Principle

**Layer order = children array order, NOT Y coordinate.**

Figma's children array is already sorted by visual stacking order:
- First child (index 0) = bottom layer (rendered first)
- Last child (index n-1) = top layer (rendered last, visually on top)

## Why NOT Y Coordinate?

Y coordinate determines vertical position, not layer order:
- A button at Y=100 could be ABOVE an image at Y=50
- Only the children array order determines visual stacking
- Figma's layer panel order is authoritative

**Example:**
- Background: Y=0 (full screen) + Layer Panel BOTTOM → zIndex 100
- PageControl: Y=60 (top of screen) + Layer Panel TOP → zIndex 300

## Extracting Layer Order

```typescript
// From Figma node children array
const layerOrder = node.children.map((child, index) => ({
  name: child.name,
  zIndex: (index + 1) * 100,  // 100, 200, 300...
  position: getPositionContext(child)  // top/center/bottom
}));
```

## zIndex Assignment Rules

| Array Index | zIndex | Rendering |
|-------------|--------|-----------|
| 0 (first) | 100 | Bottom (back) |
| 1 | 200 | Middle |
| n-1 (last) | n * 100 | Top (front) |

**Note:** Multiplying by 100 leaves room for intermediate layers (150, 250, etc.)

## Spec Output Format

```yaml
layerOrder:
  - layer: Background (zIndex: 100)
  - layer: ContentCard (zIndex: 200)
  - layer: FloatingButton (zIndex: 300)
```

## Framework-Specific Rendering

| Framework | Order Rule | Code Pattern |
|-----------|------------|--------------|
| React/HTML | Last element = top (CSS stacking) | Render highest zIndex last |
| SwiftUI ZStack | Last element = top | Sort by zIndex ascending |
| CSS z-index | Higher value = top | Explicit z-index property |

### React Example

```jsx
// Render in zIndex ascending order (lowest first)
<div className="relative">
  <Background />      {/* zIndex 100 - renders first (bottom) */}
  <HeroImage />       {/* zIndex 500 - middle */}
  <PageControl />     {/* zIndex 900 - renders last (top) */}
</div>
```

### SwiftUI Example

```swift
ZStack {
    Background()      // zIndex 100 - first = bottom
    HeroImage()       // zIndex 500 - middle
    PageControl()     // zIndex 900 - last = on top
}
```

## Critical Rules

1. **Higher zIndex = renders on top**
2. **Never sort by Y coordinate** - Use Figma children array order
3. **Always query ALL nodes** - Layer order matters for overlays, headers, FABs
4. **Missing layerOrder?** - Use document order as fallback
5. **Same zIndex?** - Components can render in any relative order

## Platform-Specific z-index Implementation

### CSS

```css
/* Stacking context is created by position + z-index */
.container {
  position: relative; /* Establishes stacking context */
}

.background {
  position: absolute;
  z-index: 100;
}

.content {
  position: relative;
  z-index: 200;
}

.floating-button {
  position: absolute;
  z-index: 300;
}
```

**Stacking context rules:**
- `z-index` only works on positioned elements (`relative`, `absolute`, `fixed`, `sticky`)
- A new stacking context is created by: `position` + `z-index`, `opacity < 1`, `transform`, `filter`, `isolation: isolate`
- Children cannot escape their parent's stacking context
- Without explicit `z-index`, elements stack in document order (later = on top)

### Tailwind CSS

Tailwind provides `z-*` utilities for z-index values:

```html
<div class="relative">
  <div class="absolute z-0">Background layer</div>
  <div class="relative z-10">Content layer</div>
  <div class="absolute z-20">Overlay layer</div>
  <div class="absolute z-50">Modal / top layer</div>
</div>
```

**Built-in z-index utilities:**

| Utility | CSS Output | Use Case |
|---------|-----------|----------|
| `z-0` | `z-index: 0` | Base layer |
| `z-10` | `z-index: 10` | Slightly elevated |
| `z-20` | `z-index: 20` | Card overlays |
| `z-30` | `z-index: 30` | Dropdowns |
| `z-40` | `z-index: 40` | Fixed headers |
| `z-50` | `z-index: 50` | Modals, top-level overlays |
| `z-auto` | `z-index: auto` | Default browser behavior |

For custom values, use arbitrary syntax: `z-[100]`, `z-[200]`, `z-[300]`.

### SwiftUI

```swift
// ZStack natural ordering: first child = bottom, last child = top
ZStack {
    BackgroundView()   // bottom layer
    ContentView()      // middle layer
    FloatingButton()   // top layer
}

// Explicit .zIndex() modifier to override natural ordering
ZStack {
    ContentView()
        .zIndex(1)      // middle
    BackgroundView()
        .zIndex(0)      // bottom (even though listed first)
    FloatingButton()
        .zIndex(2)      // top
}
```

**SwiftUI z-index rules:**
- Default `zIndex` is `0` for all views
- Higher values render on top
- Natural ordering (source order) breaks ties
- `.zIndex()` only affects siblings within the same container

## Absolute Positioning for Overlapping Elements

When Figma layers overlap (same or similar coordinates), use absolute positioning to replicate the layout.

### CSS

```css
/* Parent must be positioned for absolute children */
.card {
  position: relative;
  width: 354px;
  height: 200px;
}

/* Overlay positioned within parent */
.card-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;   /* Full coverage */
  z-index: 200;
}

/* Badge in top-right corner */
.badge {
  position: absolute;
  top: -8px;
  right: -8px;
  z-index: 300;
}

/* Bottom-aligned content */
.footer-content {
  position: absolute;
  bottom: 16px;
  left: 16px;
  right: 16px;
  z-index: 200;
}
```

### Tailwind CSS

```html
<!-- Full overlay on image -->
<div class="relative w-[354px] h-[200px]">
  <img class="absolute inset-0 w-full h-full object-cover" src="bg.png" />
  <div class="absolute inset-0 bg-black/50 z-10"></div>
  <p class="absolute bottom-4 left-4 text-white z-20">Overlay text</p>
</div>

<!-- Floating badge -->
<div class="relative">
  <div class="rounded-lg bg-white p-4">Card content</div>
  <span class="absolute -top-2 -right-2 bg-red-500 text-white text-xs rounded-full px-2 py-0.5 z-10">
    New
  </span>
</div>

<!-- Stacked cards -->
<div class="relative h-[300px]">
  <div class="absolute top-0 left-0 w-full bg-white rounded-lg shadow-md z-0">Back card</div>
  <div class="absolute top-4 left-4 w-full bg-white rounded-lg shadow-lg z-10">Front card</div>
</div>
```

**Tailwind inset utilities:**

| Utility | CSS | Description |
|---------|-----|-------------|
| `inset-0` | `top:0; right:0; bottom:0; left:0` | Full parent coverage |
| `inset-x-0` | `left:0; right:0` | Full width |
| `inset-y-0` | `top:0; bottom:0` | Full height |
| `top-4` | `top: 1rem` | Offset from top |
| `bottom-4` | `bottom: 1rem` | Offset from bottom |
| `-top-2` | `top: -0.5rem` | Negative offset (overflow) |

### SwiftUI

```swift
// Full overlay using .overlay()
Image("background")
    .resizable()
    .aspectRatio(contentMode: .fill)
    .frame(width: 354, height: 200)
    .overlay(
        LinearGradient(
            colors: [.clear, .black.opacity(0.6)],
            startPoint: .top,
            endPoint: .bottom
        )
    )
    .overlay(
        Text("Overlay text")
            .foregroundColor(.white),
        alignment: .bottomLeading
    )

// ZStack for full overlap control
ZStack(alignment: .topTrailing) {
    // Base card
    RoundedRectangle(cornerRadius: 12)
        .fill(Color.white)
        .frame(width: 200, height: 120)

    // Floating badge
    Text("New")
        .font(.caption2)
        .foregroundColor(.white)
        .padding(.horizontal, 8)
        .padding(.vertical, 2)
        .background(Color.red)
        .cornerRadius(8)
        .offset(x: 8, y: -8)
}

// Stacked cards with offset
ZStack {
    RoundedRectangle(cornerRadius: 12)
        .fill(Color.white)
        .shadow(radius: 4)
        .zIndex(0)

    RoundedRectangle(cornerRadius: 12)
        .fill(Color.white)
        .shadow(radius: 8)
        .offset(x: 8, y: 8)
        .zIndex(1)
}
```

## Common Overlay Patterns

### Pattern 1: Text on Image

Overlay text directly on a background image with a gradient scrim for readability.

**CSS/Tailwind:**
```html
<div class="relative">
  <img class="w-full h-48 object-cover rounded-lg" src="hero.png" />
  <div class="absolute inset-0 bg-gradient-to-t from-black/60 to-transparent rounded-lg"></div>
  <h2 class="absolute bottom-4 left-4 text-white text-xl font-bold z-10">Title</h2>
</div>
```

**SwiftUI:**
```swift
Image("hero")
    .resizable()
    .aspectRatio(contentMode: .fill)
    .frame(height: 192)
    .clipped()
    .overlay(
        LinearGradient(colors: [.clear, .black.opacity(0.6)],
                       startPoint: .top, endPoint: .bottom)
    )
    .overlay(
        Text("Title")
            .font(.title2).bold()
            .foregroundColor(.white)
            .padding(),
        alignment: .bottomLeading
    )
    .cornerRadius(12)
```

### Pattern 2: Floating Badge

A notification badge or label positioned outside the parent boundary.

**CSS/Tailwind:**
```html
<div class="relative inline-block">
  <div class="w-10 h-10 bg-gray-200 rounded-full"></div>
  <span class="absolute -top-1 -right-1 w-5 h-5 bg-red-500 text-white text-xs
               rounded-full flex items-center justify-center z-10">3</span>
</div>
```

**SwiftUI:**
```swift
ZStack(alignment: .topTrailing) {
    Circle()
        .fill(Color.gray.opacity(0.2))
        .frame(width: 40, height: 40)

    Text("3")
        .font(.caption2)
        .foregroundColor(.white)
        .frame(width: 20, height: 20)
        .background(Color.red)
        .clipShape(Circle())
        .offset(x: 4, y: -4)
}
```

### Pattern 3: Stacked Cards

Cards arranged with slight offset to create a depth/stack effect.

**CSS/Tailwind:**
```html
<div class="relative h-[220px] w-[300px]">
  <div class="absolute top-4 left-4 w-[280px] h-[180px] bg-white rounded-xl shadow z-0"></div>
  <div class="absolute top-2 left-2 w-[280px] h-[180px] bg-white rounded-xl shadow-md z-10"></div>
  <div class="absolute top-0 left-0 w-[280px] h-[180px] bg-white rounded-xl shadow-lg z-20">
    Front card content
  </div>
</div>
```

**SwiftUI:**
```swift
ZStack {
    RoundedRectangle(cornerRadius: 16)
        .fill(Color.white)
        .frame(width: 280, height: 180)
        .shadow(radius: 2)
        .offset(x: 8, y: 8)
        .zIndex(0)

    RoundedRectangle(cornerRadius: 16)
        .fill(Color.white)
        .frame(width: 280, height: 180)
        .shadow(radius: 4)
        .offset(x: 4, y: 4)
        .zIndex(1)

    RoundedRectangle(cornerRadius: 16)
        .fill(Color.white)
        .frame(width: 280, height: 180)
        .shadow(radius: 6)
        .zIndex(2)
}
```
