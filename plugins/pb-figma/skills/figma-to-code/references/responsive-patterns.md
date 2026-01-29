# Responsive Patterns Reference

## Breakpoint Definitions

Standard Figma frame sizes mapped to platform breakpoints:

| Breakpoint | Width | Figma Frame | Tailwind Prefix | CSS Query | SwiftUI |
|------------|-------|-------------|-----------------|-----------|---------|
| Mobile (default) | 375px | iPhone SE / small phones | None (base) | Default styles | `.compact` sizeClass |
| Tablet | 768px | iPad Mini / tablets | `md:` | `@media (min-width: 768px)` | `.regular` sizeClass |
| Desktop | 1440px | Standard desktop | `lg:` / `xl:` | `@media (min-width: 1024px)` | N/A (web only) |

## Mobile-First Approach

All responsive code MUST follow mobile-first methodology:

1. **Base styles target 375px (mobile)** -- no prefix, no media query
2. **Progressive enhancement** adds styles for larger screens
3. **Desktop-only components cannot receive PASS status** in compliance checks

```
WRONG:  Define desktop layout, then override for mobile
RIGHT:  Define mobile layout, then enhance for tablet/desktop
```

## Figma Constraints to Responsive Code

| Figma Constraint | Responsive Intent | Tailwind | CSS |
|------------------|-------------------|----------|-----|
| Fill container (width) | Fluid width | `w-full` | `width: 100%` |
| Fixed width | Static size | `w-[Xpx]` | `width: Xpx` |
| Hug contents | Shrink-wrap | `w-auto` or `w-fit` | `width: auto` |
| Min/Max width | Responsive range | `min-w-[X] max-w-[Y]` | `min-width: X; max-width: Y` |
| constraints: STRETCH | Fill parent | `w-full` or `max-w-full` | `width: 100%` |
| constraints: MIN | Hug contents | No width constraint | `width: fit-content` |

**Responsive hints from design-analyst:**

- `constraints.horizontal: STRETCH` AND no `maxWidth` --> `Responsive: content stretches to fill parent`
- `maxWidth` set --> `Responsive: max width {maxWidth}pt`

## Platform-Specific Responsive Patterns

### Tailwind CSS (React)

Use responsive prefixes. Mobile styles are the unprefixed base.

```tsx
// Card that adapts to screen size
<article className={cn(
  // Mobile (default): Full width, vertical stack
  "flex flex-col w-full gap-4 p-4",
  // Tablet: Horizontal layout, constrained width
  "md:flex-row md:max-w-[600px] md:p-6",
  // Desktop: Larger padding
  "lg:max-w-[800px] lg:p-8"
)}>
  <Image
    src={imageSrc}
    alt={imageAlt}
    className="w-full h-48 md:w-1/3 md:h-auto object-cover rounded-lg"
  />
  <div className="flex-1">
    <h2 className="text-lg md:text-xl lg:text-2xl font-bold">{title}</h2>
    <p className="text-sm md:text-base text-gray-600">{description}</p>
  </div>
</article>
```

**Breakpoint mapping:**

| Figma Frame | Tailwind Breakpoint | Use |
|-------------|---------------------|-----|
| 375px (Mobile) | Default (no prefix) | Mobile-first base |
| 768px (Tablet) | `md:` | Tablet layouts |
| 1024px (Desktop) | `lg:` | Desktop layouts |
| 1280px (Large) | `xl:` | Wide screens |

### CSS Media Queries

When Tailwind is not available, use standard `@media` queries:

```css
/* Mobile-first base */
.card {
  display: flex;
  flex-direction: column;
  width: 100%;
  gap: 1rem;
  padding: 1rem;
}

/* Tablet */
@media (min-width: 768px) {
  .card {
    flex-direction: row;
    max-width: 600px;
    padding: 1.5rem;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .card {
    max-width: 800px;
    padding: 2rem;
  }
}
```

### SwiftUI

SwiftUI uses adaptive layout, size classes, and geometry-based approaches.

**Rule 1 -- Content Width Cap (iPad readability):**

```swift
VStack(spacing: 16) {
    // content
}
.frame(maxWidth: 600)        // prevent over-stretching on iPad
.frame(maxWidth: .infinity)  // center within parent
```

**Rule 2 -- Adaptive Grid (3+ repeating cards):**

```swift
private let columns = [GridItem(.adaptive(minimum: 280, maximum: 400))]

var body: some View {
    LazyVGrid(columns: columns, spacing: 16) {
        ForEach(items) { item in
            CardView(item: item)
        }
    }
}
```

**Rule 3 -- Safe Layout Defaults:**

```swift
// DO: Use flexible widths
.frame(maxWidth: .infinity)

// DON'T: Use screen-dependent widths
.frame(width: UIScreen.main.bounds.width)   // breaks on iPad/landscape
.frame(width: 393)                          // hardcoded iPhone width

// DO: Use horizontal padding for edge spacing
.padding(.horizontal, 16)

// DON'T: Calculate padding from screen width
.padding(.horizontal, (UIScreen.main.bounds.width - 361) / 2)
```

**Rule 4 -- Size Class (complex layouts only):**

Use `horizontalSizeClass` only when spec explicitly mentions different tablet layout:

```swift
@Environment(\.horizontalSizeClass) var horizontalSizeClass

var body: some View {
    if horizontalSizeClass == .regular {
        // iPad: side-by-side layout
        HStack(spacing: 24) {
            leftContent
            rightContent
        }
    } else {
        // iPhone: stacked layout
        VStack(spacing: 16) {
            leftContent
            rightContent
        }
    }
}
```

## Common Responsive Patterns

### Stack Direction Change (Vertical to Horizontal)

| Platform | Mobile | Tablet/Desktop |
|----------|--------|----------------|
| Tailwind | `flex flex-col` | `md:flex-row` |
| CSS | `flex-direction: column` | `@media (min-width: 768px) { flex-direction: row }` |
| SwiftUI | `VStack` | `horizontalSizeClass == .regular ? HStack : VStack` |

### Grid Column Count Change

| Platform | Mobile | Tablet | Desktop |
|----------|--------|--------|---------|
| Tailwind | `grid-cols-1` | `md:grid-cols-2` | `lg:grid-cols-3` |
| CSS | `grid-template-columns: 1fr` | `@media ... { grid-template-columns: repeat(2, 1fr) }` | `@media ... { repeat(3, 1fr) }` |
| SwiftUI | `LazyVGrid(.adaptive(minimum: 280))` | Auto-adjusts columns | Auto-adjusts columns |

```tsx
// Tailwind grid example
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {items.map(item => <Card key={item.id} {...item} />)}
</div>
```

### Font Size Scaling

| Platform | Mobile | Tablet | Desktop |
|----------|--------|--------|---------|
| Tailwind | `text-lg` | `md:text-xl` | `lg:text-2xl` |
| CSS | `font-size: 1.125rem` | `@media ... { font-size: 1.25rem }` | `@media ... { font-size: 1.5rem }` |
| SwiftUI | `.font(.title3)` | Dynamic Type handles scaling | -- |

### Spacing and Padding Adjustments

| Platform | Mobile | Tablet | Desktop |
|----------|--------|--------|---------|
| Tailwind | `p-4 gap-4` | `md:p-6 md:gap-6` | `lg:p-8 lg:gap-8` |
| CSS | `padding: 1rem` | `@media ... { padding: 1.5rem }` | `@media ... { padding: 2rem }` |
| SwiftUI | `.padding(16)` | `.padding(24)` via sizeClass | -- |

### Container Pattern

```tsx
// Responsive container with max-width
<div className="container mx-auto px-4 md:px-6 lg:px-8">
  {children}
</div>

// Or with explicit max-widths
<div className="w-full max-w-7xl mx-auto px-4">
  {children}
</div>
```

### Show/Hide Elements by Breakpoint

```tsx
// Tailwind: hidden on mobile, visible on tablet+
<div className="hidden md:block">Desktop sidebar</div>

// Tailwind: visible on mobile, hidden on tablet+
<div className="block md:hidden">Mobile menu button</div>
```

```css
/* CSS equivalent */
.desktop-only { display: none; }
@media (min-width: 768px) { .desktop-only { display: block; } }

.mobile-only { display: block; }
@media (min-width: 768px) { .mobile-only { display: none; } }
```

## SwiftUI Common Mistakes

| Mistake | Fix |
|---------|-----|
| Hardcoding `UIScreen.main.bounds.width` | Use `.frame(maxWidth: .infinity)` |
| Missing maxWidth cap on root container | Add `.frame(maxWidth: 600).frame(maxWidth: .infinity)` |
| Using VStack for 5+ repeating cards | Use `LazyVGrid(.adaptive(...))` |
| Hardcoded iPhone width (e.g., 393) | Use flexible `.frame(maxWidth:)` |

## Auto Layout to Flexbox/Grid Mapping

| Figma Auto Layout | Tailwind |
|-------------------|----------|
| Horizontal | `flex flex-row` |
| Vertical | `flex flex-col` |
| Wrap | `flex flex-wrap` |
| Gap: 16 | `gap-4` |
| Padding: 24 | `p-6` |
| Space between | `justify-between` |
| Align center | `items-center` |

## Cross-References

- **Testing procedures:** `@references/responsive-validation.md` -- covers responsive compliance test procedures, verification checklists, and pass/fail criteria
- **Layout tokens:** `@references/token-mapping.md` -- spacing and layout token mappings
