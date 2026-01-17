# Accessibility Guide

WCAG 2.1 compliant patterns for accessible React applications. Includes Web Interface Guidelines from Vercel Labs.

---

## Critical Rules

| Rule | Requirement |
|------|-------------|
| Icon buttons | MUST have `aria-label` |
| Interactive elements | Use `button` for actions, `a/Link` for navigation |
| Images | MUST have `alt` (or `alt=""` if decorative) |
| Form inputs | MUST have `autocomplete` attribute |
| Headings | MUST follow hierarchy (h1 → h2 → h3) |
| Errors | MUST appear inline next to fields |
| Submit buttons | Stay enabled; show spinner during request |
| Long lists | Virtualize if >50 items |
| Animations | Honor `prefers-reduced-motion` |

---

## 1. Semantic HTML

### Use Correct Elements

```typescript
// ❌ WRONG - Non-semantic
<div onClick={handleClick}>Click me</div>
<div className="heading">Title</div>
<span onClick={navigate}>Go to page</span>

// ✅ CORRECT - Semantic
<button onClick={handleClick}>Click me</button>
<h2>Title</h2>
<Link href="/page">Go to page</Link>
```

### Button vs Link

```typescript
// Actions (triggers functionality) → button
<button onClick={() => setOpen(true)}>Open Modal</button>
<button onClick={handleSubmit}>Submit</button>
<button onClick={handleDelete}>Delete</button>

// Navigation (changes URL) → a/Link
<Link href="/dashboard">Go to Dashboard</Link>
<a href="/download.pdf" download>Download PDF</a>
<Link href="/profile">View Profile</Link>
```

### Heading Hierarchy

```typescript
// ❌ WRONG - Skipped heading level
<h1>Page Title</h1>
<h3>Section Title</h3>  // Skipped h2!

// ✅ CORRECT - Proper hierarchy
<h1>Page Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>
```

---

## 2. ARIA Attributes

### Icon Buttons

```typescript
// ❌ WRONG - No accessible name
<button>
    <XIcon />
</button>

<button>
    <TrashIcon />
</button>

// ✅ CORRECT - With aria-label
<button aria-label="Close dialog">
    <XIcon />
</button>

<button aria-label="Delete item">
    <TrashIcon />
</button>
```

### States and Properties

```typescript
// Expanded/collapsed
<button
    aria-expanded={isOpen}
    aria-controls="menu-content"
    onClick={() => setIsOpen(!isOpen)}
>
    Menu
</button>
<div id="menu-content" hidden={!isOpen}>
    {/* Menu items */}
</div>

// Selected state
<button
    aria-selected={isSelected}
    role="tab"
    onClick={() => setSelected(true)}
>
    Tab 1
</button>

// Disabled state (visible but inactive)
<button
    aria-disabled={isDisabled}
    onClick={isDisabled ? undefined : handleClick}
    className={isDisabled ? 'opacity-50 cursor-not-allowed' : ''}
>
    Submit
</button>

// Loading state
<button aria-busy={isLoading} disabled={isLoading}>
    {isLoading ? <Loader2 className="animate-spin" /> : 'Submit'}
</button>
```

### Live Regions

```typescript
// For dynamic content updates
<div aria-live="polite" aria-atomic="true">
    {statusMessage}
</div>

// For urgent announcements
<div aria-live="assertive" role="alert">
    {errorMessage}
</div>
```

---

## 3. Keyboard Navigation

### Focus Management

```typescript
import { useRef, useEffect } from 'react';

// Auto-focus on mount
export const Modal: React.FC<{ onClose: () => void }> = ({ onClose }) => {
    const closeButtonRef = useRef<HTMLButtonElement>(null);

    useEffect(() => {
        closeButtonRef.current?.focus();
    }, []);

    return (
        <Dialog>
            <button ref={closeButtonRef} onClick={onClose}>
                Close
            </button>
        </Dialog>
    );
};
```

### Tab Order

```typescript
// Control tab order with tabIndex
<div>
    <button tabIndex={0}>First</button>
    <button tabIndex={0}>Second</button>
    <button tabIndex={-1}>Skip (programmatically focusable)</button>
</div>
```

### Keyboard Handlers

```typescript
// ❌ WRONG - Only click handler
<div onClick={handleSelect}>Option 1</div>

// ✅ CORRECT - Keyboard support
<div
    role="option"
    tabIndex={0}
    onClick={handleSelect}
    onKeyDown={(e) => {
        if (e.key === 'Enter' || e.key === ' ') {
            e.preventDefault();
            handleSelect();
        }
    }}
>
    Option 1
</div>

// Or better - use button
<button onClick={handleSelect}>Option 1</button>
```

### Focus Trap for Modals

```typescript
import { useEffect, useRef } from 'react';

function useFocusTrap(isActive: boolean) {
    const containerRef = useRef<HTMLDivElement>(null);

    useEffect(() => {
        if (!isActive) return;

        const container = containerRef.current;
        if (!container) return;

        const focusableElements = container.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );

        const firstElement = focusableElements[0] as HTMLElement;
        const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;

        const handleKeyDown = (e: KeyboardEvent) => {
            if (e.key !== 'Tab') return;

            if (e.shiftKey && document.activeElement === firstElement) {
                e.preventDefault();
                lastElement.focus();
            } else if (!e.shiftKey && document.activeElement === lastElement) {
                e.preventDefault();
                firstElement.focus();
            }
        };

        container.addEventListener('keydown', handleKeyDown);
        firstElement?.focus();

        return () => container.removeEventListener('keydown', handleKeyDown);
    }, [isActive]);

    return containerRef;
}
```

---

## 4. Forms

### Labels and Inputs

```typescript
// ❌ WRONG - No label association
<label>Email</label>
<input type="email" />

// ✅ CORRECT - Explicit association
<label htmlFor="email">Email</label>
<input id="email" type="email" name="email" autoComplete="email" />

// Or implicit
<label>
    Email
    <input type="email" name="email" autoComplete="email" />
</label>
```

### Autocomplete Attribute

```typescript
// Always include autocomplete for user data
<input type="text" name="name" autoComplete="name" />
<input type="email" name="email" autoComplete="email" />
<input type="tel" name="phone" autoComplete="tel" />
<input type="text" name="address" autoComplete="street-address" />
<input type="text" name="city" autoComplete="address-level2" />
<input type="text" name="zip" autoComplete="postal-code" />
<input type="password" name="password" autoComplete="current-password" />
<input type="password" name="newPassword" autoComplete="new-password" />
```

### Error Messages

```typescript
// ❌ WRONG - Error not associated
<input id="email" />
{error && <span className="text-red-500">Invalid email</span>}

// ✅ CORRECT - Error linked with aria-describedby
<input
    id="email"
    aria-invalid={!!error}
    aria-describedby={error ? 'email-error' : undefined}
/>
{error && (
    <span id="email-error" className="text-sm text-destructive" role="alert">
        {error}
    </span>
)}
```

### Required Fields

```typescript
<label htmlFor="name">
    Name <span aria-hidden="true">*</span>
</label>
<input
    id="name"
    required
    aria-required="true"
/>
```

### Submit Button Pattern

```typescript
// ❌ WRONG - Disabled button
<Button type="submit" disabled={isSubmitting}>
    Submit
</Button>

// ✅ CORRECT - Enabled with spinner
<Button type="submit" aria-busy={isSubmitting}>
    {isSubmitting ? (
        <>
            <Loader2 className="mr-2 h-4 w-4 animate-spin" />
            Submitting...
        </>
    ) : (
        'Submit'
    )}
</Button>
```

---

## 5. Images and Media

### Alt Text

```typescript
// Informative image
<img src="/product.jpg" alt="Red running shoes, side view" />

// Decorative image (screen readers skip)
<img src="/decorative-pattern.svg" alt="" aria-hidden="true" />

// Image with text
<img src="/logo.svg" alt="Acme Company" />

// Complex image with description
<figure>
    <img src="/chart.png" alt="Sales growth chart" aria-describedby="chart-desc" />
    <figcaption id="chart-desc">
        Chart showing 25% sales growth from Q1 to Q4 2024
    </figcaption>
</figure>
```

### Icons

```typescript
// Decorative icon (with text)
<button>
    <PlusIcon aria-hidden="true" />
    <span>Add Item</span>
</button>

// Functional icon (standalone)
<button aria-label="Add item">
    <PlusIcon />
</button>
```

---

## 6. Color and Contrast

### Don't Rely on Color Alone

```typescript
// ❌ WRONG - Only color indicates state
<span className={isError ? 'text-red-500' : 'text-green-500'}>
    {message}
</span>

// ✅ CORRECT - Icon + color + text
<span className={isError ? 'text-red-500' : 'text-green-500'}>
    {isError ? <XCircle className="inline mr-1" /> : <CheckCircle className="inline mr-1" />}
    {isError ? 'Error: ' : 'Success: '}
    {message}
</span>
```

### Focus Indicators

```typescript
// Never remove focus outline without replacement
// ❌ WRONG
button:focus {
    outline: none;
}

// ✅ CORRECT - Custom but visible
button:focus-visible {
    outline: 2px solid var(--color-primary);
    outline-offset: 2px;
}
```

---

## 7. Motion and Animation

### Respect Motion Preferences

```typescript
// CSS approach
@media (prefers-reduced-motion: reduce) {
    * {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}

// React hook approach
function usePrefersReducedMotion() {
    const [prefersReduced, setPrefersReduced] = useState(false);

    useEffect(() => {
        const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
        setPrefersReduced(mediaQuery.matches);

        const handler = (e: MediaQueryListEvent) => setPrefersReduced(e.matches);
        mediaQuery.addEventListener('change', handler);
        return () => mediaQuery.removeEventListener('change', handler);
    }, []);

    return prefersReduced;
}

// Usage
const prefersReduced = usePrefersReducedMotion();

<motion.div
    animate={{ x: 100 }}
    transition={{ duration: prefersReduced ? 0 : 0.3 }}
/>
```

---

## 8. Long Lists

### Virtualization

```typescript
// For lists with >50 items
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Item[] }) {
    const parentRef = useRef<HTMLDivElement>(null);

    const virtualizer = useVirtualizer({
        count: items.length,
        getScrollElement: () => parentRef.current,
        estimateSize: () => 50,
    });

    return (
        <div ref={parentRef} className="h-96 overflow-auto">
            <div
                style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}
            >
                {virtualizer.getVirtualItems().map((virtualItem) => (
                    <div
                        key={virtualItem.key}
                        style={{
                            position: 'absolute',
                            top: 0,
                            left: 0,
                            width: '100%',
                            transform: `translateY(${virtualItem.start}px)`,
                        }}
                    >
                        <ListItem item={items[virtualItem.index]} />
                    </div>
                ))}
            </div>
        </div>
    );
}
```

---

## 9. Testing

### Automated Tests

```typescript
import { render, screen } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('MyComponent accessibility', () => {
    it('has no accessibility violations', async () => {
        const { container } = render(<MyComponent />);
        const results = await axe(container);
        expect(results).toHaveNoViolations();
    });

    it('has accessible button', () => {
        render(<IconButton />);
        expect(screen.getByRole('button', { name: /close/i })).toBeInTheDocument();
    });

    it('form has proper labels', () => {
        render(<LoginForm />);
        expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
        expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    });
});
```

### Manual Checks

1. **Keyboard**: Navigate entire page with Tab, Shift+Tab, Enter, Space, Escape
2. **Screen Reader**: Test with VoiceOver (Mac) or NVDA (Windows)
3. **Zoom**: Test at 200% zoom
4. **Color**: Check with color blindness simulators
5. **Focus**: Verify focus is always visible

---

## Quick Reference

| Element | Required Attributes |
|---------|---------------------|
| `<button>` (icon only) | `aria-label` |
| `<img>` | `alt` |
| `<input>` | `id`, `name`, `autocomplete`, associated `<label>` |
| `<input>` with error | `aria-invalid`, `aria-describedby` |
| Modal/dialog | Focus trap, `aria-modal`, `role="dialog"` |
| Loading button | `aria-busy` |
| Toggle | `aria-pressed` or `aria-expanded` |
| Tab | `role="tab"`, `aria-selected` |

---

## Accessibility Checklist

- [ ] All interactive elements are focusable and have visible focus indicator
- [ ] All images have appropriate alt text
- [ ] All form inputs have associated labels
- [ ] Icon-only buttons have aria-label
- [ ] Color is not the only means of conveying information
- [ ] Headings follow proper hierarchy
- [ ] Page is navigable by keyboard alone
- [ ] Motion respects prefers-reduced-motion
- [ ] Error messages are associated with inputs
- [ ] Long lists are virtualized
