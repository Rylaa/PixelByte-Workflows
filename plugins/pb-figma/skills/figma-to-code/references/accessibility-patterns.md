# Accessibility Patterns Reference (Code Generation)

Patterns and rules for generating accessible code from Figma designs. This reference covers **what to generate** -- the accessibility modifiers, attributes, and semantic structures that code generators must produce.

> **Cross-reference:** See `@references/accessibility-validation.md` for the compliance-checker validation checklists (how to verify accessibility after generation).

---

## Semantic HTML Elements (React)

Map Figma component purposes to proper semantic HTML elements instead of generic `<div>` wrappers.

| Figma Component Purpose | Semantic Element | When to Use |
|------------------------|-----------------|-------------|
| Page section | `<section>` | Thematic grouping of content |
| Self-contained content | `<article>` | Blog post, card, comment |
| Navigation links | `<nav>` | Primary/secondary navigation |
| Page header | `<header>` | Top-level or section header |
| Page footer | `<footer>` | Bottom-level or section footer |
| Primary content | `<main>` | Main content area (one per page) |
| Sidebar / tangential | `<aside>` | Related but separate content |
| Clickable action | `<button>` | Buttons, toggles, actions |
| Navigational link | `<a>` | Links to other pages/resources |
| Form fields | `<input>`, `<select>`, `<textarea>` | User input elements |
| List of items | `<ul>` / `<ol>` | Repeated similar items |

### Semantic HTML Example

```tsx
// WRONG - div soup
<div onClick={handleClick}>Click me</div>

// CORRECT - semantic element
<button type="button" onClick={handleClick}>Click me</button>
```

**Compliance check:** Excessive `<div>` elements without roles indicate missing semantic structure.

```bash
# Should NOT find excessive divs without roles
Grep("<div(?![^>]*role=)", path="{component_file_path}")
```

---

## ARIA Attributes (React)

### Core ARIA Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `aria-label` | Accessible name for elements without visible text | `aria-label="Close dialog"` |
| `aria-describedby` | Points to element providing additional description | `aria-describedby="error-msg"` |
| `aria-hidden` | Hides decorative elements from screen readers | `aria-hidden="true"` |
| `role` | Defines semantic role when HTML element is insufficient | `role="tablist"` |
| `aria-disabled` | Indicates disabled state accessibly | `aria-disabled={disabled}` |
| `aria-expanded` | Indicates expandable element state | `aria-expanded={isOpen}` |
| `aria-labelledby` | Points to element providing the accessible name | `aria-labelledby="heading-id"` |

### React Accessibility Pattern

```tsx
<button
    type="button"
    aria-label={ariaLabel}
    aria-disabled={disabled}
    className="... focus:outline-none focus:ring-2 focus:ring-primary focus:ring-offset-2"
>
    {children}
</button>
```

### Focus Management

All interactive elements must include focus-visible styles:

```tsx
// Tailwind focus pattern
className="focus:outline-none focus:ring-2 focus:ring-primary focus:ring-offset-2"

// Or focus-visible for keyboard-only users
className="focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-primary"
```

### Keyboard Navigation

- All interactive elements must be reachable via Tab key
- Use `tabIndex={0}` for custom interactive elements that are not natively focusable
- Use `tabIndex={-1}` for programmatically focusable elements (e.g., modals)
- Never use positive `tabIndex` values

---

## SwiftUI Accessibility

### Core Modifiers

| Modifier | Purpose | Example |
|----------|---------|---------|
| `.accessibilityLabel()` | VoiceOver spoken name | `.accessibilityLabel("Submit form")` |
| `.accessibilityHint()` | Additional context for action | `.accessibilityHint("Double tap to submit the form")` |
| `.accessibilityAddTraits()` | Semantic meaning | `.accessibilityAddTraits(.isButton)` |
| `.accessibilityElement(children:)` | Group child elements | `.accessibilityElement(children: .combine)` |
| `.accessibilityHidden()` | Hide decorative elements | `.accessibilityHidden(true)` |
| `.accessibilityValue()` | Current value of a control | `.accessibilityValue("\(progress)%")` |

### VoiceOver Patterns

#### Button with full accessibility

```swift
Button("Submit") {
    submitAction()
}
.accessibilityLabel("Submit form")
.accessibilityHint("Double tap to submit the form")
.accessibilityAddTraits(.isButton)
```

#### Grouped content (card pattern)

```swift
VStack {
    Text(title)
        .font(.headline)
    Text(description)
        .font(.body)
        .foregroundColor(Color("TextSecondary"))
}
.accessibilityElement(children: .combine)
.accessibilityLabel(accessibilityDescription)
```

With a computed accessibility description:

```swift
private var accessibilityDescription: String {
    var desc = "Card: \(title)"
    if let description = description {
        desc += ". \(description)"
    }
    return desc
}
```

#### Image with accessibility

```swift
// Informational image
Image("growth-chart")
    .resizable()
    .aspectRatio(contentMode: .fit)
    .accessibilityLabel("Projected Growth chart showing upward trend")

// Decorative image (hidden from VoiceOver)
Image("decorative-bg")
    .resizable()
    .accessibilityHidden(true)
```

### Dynamic Type Support

Use system fonts to automatically support Dynamic Type:

```swift
// CORRECT - system fonts scale with Dynamic Type
Text("Title")
    .font(.system(size: 16, weight: .semibold))

Text("Body")
    .font(.body)

// For explicit control
Text("Title")
    .dynamicTypeSize(.medium ... .xxxLarge)
```

**Self-check:**
- [ ] Uses system font sizes or `.dynamicTypeSize` modifier
- [ ] No fixed-height containers that would clip scaled text

---

## Image Accessibility

### Decision Table

| Image Type | React | SwiftUI |
|------------|-------|---------|
| Informational image | `alt="Descriptive text"` | `.accessibilityLabel("Descriptive text")` |
| Decorative image | `alt=""` + `aria-hidden="true"` | `.accessibilityHidden(true)` |
| Icon with meaning | `alt="Action name"` or `aria-label` on parent | `.accessibilityLabel("Action name")` |
| Image with embedded text | `alt` includes text content | `.accessibilityLabel()` includes text content |
| Icon next to text label | `alt=""` (redundant) | `.accessibilityHidden(true)` |

### React Examples

```tsx
// Informational image
<Image
    src="/images/hero.jpg"
    alt="Hero image showing product dashboard"
    fill
    className="object-cover"
/>

// Decorative image
<img
    src="/images/pattern.svg"
    alt=""
    aria-hidden="true"
    className="absolute inset-0"
/>

// Icon with text label (icon is decorative)
<button aria-label="Settings">
    <SettingsIcon aria-hidden="true" />
    <span>Settings</span>
</button>
```

### SwiftUI Examples

```swift
// Informational image
Image("hero")
    .resizable()
    .aspectRatio(contentMode: .fill)
    .accessibilityLabel("Product dashboard showing analytics")

// Decorative image
Image("pattern-bg")
    .resizable()
    .accessibilityHidden(true)
```

---

## Color Contrast

### WCAG AA Requirements

| Text Type | Minimum Ratio | Example |
|-----------|--------------|---------|
| Normal text (< 18pt / < 14pt bold) | 4.5:1 | Body copy, labels, captions |
| Large text (>= 18pt / >= 14pt bold) | 3:1 | Headings, large buttons |
| Non-text UI components | 3:1 | Icons, borders, focus indicators |

### Compliance Checks

```bash
# React: Verify color contrast values in generated code
Grep("text-", path="{component_file_path}")
Grep("bg-", path="{component_file_path}")
```

For SwiftUI, ensure foreground/background color pairs meet contrast ratios:

```swift
// Verify contrast between text and background
Text("Label")
    .foregroundColor(Color("TextPrimary"))     // e.g., #1A1A1A
    .background(Color("CardBackground"))        // e.g., #FFFFFF
    // Contrast ratio: 16.8:1 -- PASSES AA
```

---

## Platform Accessibility Checklist

### React / Tailwind

| Check | Requirement |
|-------|-------------|
| Semantic elements | Proper HTML elements for purpose (no div soup) |
| Alt text | All images have `alt` attributes |
| ARIA labels | Interactive elements have `aria-label` or `aria-labelledby` |
| Focus states | `focus-visible` styles present for interactive elements |
| Keyboard accessible | All interactive elements reachable via Tab key |
| Color contrast | Text meets WCAG AA (>= 4.5:1 normal, >= 3:1 large) |
| Focus ring | `focus:ring-*` or `focus-visible:*` classes applied |

### SwiftUI

| Check | Requirement |
|-------|-------------|
| VoiceOver labels | All meaningful elements have `.accessibilityLabel()` |
| Hints | Interactive elements have `.accessibilityHint()` when needed |
| Traits | `.accessibilityAddTraits()` for semantic meaning |
| Dynamic Type | Uses system font sizes or `.dynamicTypeSize` modifier |
| Color contrast | Meets WCAG AA (4.5:1) |
| VoiceOver testing | Test with VoiceOver enabled |
| Grouped content | `.accessibilityElement(children: .combine)` for card-like views |
| Decorative images | Hidden from VoiceOver with `.accessibilityHidden(true)` |

---

## Compliance-Checker Integration

The compliance-checker requires all components to pass accessibility checks before receiving PASS status:

**React verification:**
- [ ] jest-axe verification -- Run automated accessibility tests with 0 violations
- [ ] Semantic elements -- Proper HTML elements for purpose
- [ ] Alt text -- All images have alt attributes
- [ ] ARIA labels -- Interactive elements have aria-label or aria-labelledby
- [ ] Focus states -- Focus-visible styles present
- [ ] Keyboard accessible -- All interactive elements reachable via Tab
- [ ] Color contrast -- WCAG AA compliance

**SwiftUI verification:**
- [ ] Accessibility -- VoiceOver labels, hints, traits
- [ ] Dynamic Type support -- Uses system font sizes or .dynamicTypeSize modifier

> **Cross-reference:** See `@references/accessibility-validation.md` for the full validation checklist and automated verification commands.
