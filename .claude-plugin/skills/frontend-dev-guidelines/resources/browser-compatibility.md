# Browser Compatibility Guide

Cross-browser compatibility, Safari quirks, and Senior Frontend Developer competencies.

---

## 1. Safari Critical Compatibility (iOS/macOS)

> **Safari is the most problematic browser.** Check Safari on every code change!

### Date/Time API

```typescript
// ❌ DOES NOT WORK in Safari
new Date('2024-01-15 10:30:00');  // Invalid Date

// ✅ Safari compatible
new Date('2024-01-15T10:30:00');  // ISO 8601 format
new Date('2024-01-15T10:30:00Z'); // UTC
new Date(2024, 0, 15, 10, 30, 0); // Constructor args
```

### CSS Flexbox/Grid

```css
/* ❌ Safari gap support missing (older versions) */
.container {
    display: flex;
    gap: 1rem;
}

/* ✅ Safari fallback */
.container {
    display: flex;
    gap: 1rem;
}
.container > * + * {
    margin-left: 1rem; /* fallback */
}

/* Or use @supports */
@supports not (gap: 1rem) {
    .container > * + * {
        margin-left: 1rem;
    }
}
```

### 100vh Issue (iOS Safari)

```css
/* ❌ iOS Safari 100vh doesn't account for address bar */
.fullscreen {
    height: 100vh;
}

/* ✅ Safari compatible */
.fullscreen {
    height: 100vh;
    height: 100dvh; /* Dynamic viewport height */
    height: -webkit-fill-available; /* iOS fallback */
}

/* Or use CSS variable */
:root {
    --app-height: 100vh;
}
@supports (height: 100dvh) {
    :root {
        --app-height: 100dvh;
    }
}
.fullscreen {
    height: var(--app-height);
}
```

### Scroll Behavior

```css
/* ❌ Safari smooth scroll can be problematic */
html {
    scroll-behavior: smooth;
}

/* ✅ Use JavaScript for controlled scroll */
```

```typescript
// ✅ Safari-safe smooth scroll
const scrollToElement = (element: HTMLElement) => {
    element.scrollIntoView({
        behavior: 'smooth',
        block: 'start',
    });
};

// Or use polyfill
// npm install smoothscroll-polyfill
```

### CSS backdrop-filter

```css
/* Safari requires -webkit prefix */
.glass {
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px); /* Safari */
}
```

### Touch Events

```typescript
// ❌ Safari passive listener warning
element.addEventListener('touchstart', handler);

// ✅ Passive listener
element.addEventListener('touchstart', handler, { passive: true });

// If scroll prevent is needed
element.addEventListener('touchmove', (e) => {
    e.preventDefault();
}, { passive: false });
```

### Input/Form Quirks

```css
/* Prevent iOS Safari input zoom */
input, select, textarea {
    font-size: 16px; /* Safari zooms below 16px */
}

/* iOS Safari input styling */
input {
    -webkit-appearance: none;
    border-radius: 0; /* Remove iOS default radius */
}

/* Safari autofill styling */
input:-webkit-autofill {
    -webkit-box-shadow: 0 0 0 30px white inset;
    -webkit-text-fill-color: inherit;
}
```

### Video/Audio

```typescript
// ❌ Safari autoplay policy
video.play(); // Rejected without user interaction

// ✅ Safari-safe autoplay
video.muted = true; // Muted autoplay works
video.playsInline = true; // Required for iOS
await video.play();
```

```html
<!-- playsInline required for iOS Safari -->
<video playsInline muted autoPlay loop>
    <source src="video.mp4" type="video/mp4" />
</video>
```

### Clipboard API

```typescript
// ❌ Safari async clipboard can be problematic
await navigator.clipboard.writeText(text);

// ✅ Safari fallback
const copyToClipboard = async (text: string) => {
    try {
        await navigator.clipboard.writeText(text);
    } catch {
        // Fallback for Safari
        const textarea = document.createElement('textarea');
        textarea.value = text;
        textarea.style.position = 'fixed';
        textarea.style.opacity = '0';
        document.body.appendChild(textarea);
        textarea.select();
        document.execCommand('copy');
        document.body.removeChild(textarea);
    }
};
```

---

## 2. Chrome-Specific Issues

### Memory Leaks

```typescript
// Use Chrome DevTools Memory tab to check
// Detached DOM nodes, event listener leaks

// ✅ Cleanup pattern
useEffect(() => {
    const handler = () => { /* ... */ };
    window.addEventListener('resize', handler);

    return () => {
        window.removeEventListener('resize', handler);
    };
}, []);
```

### Third-party Cookie Restrictions

```typescript
// Chrome restricts 3rd party cookies
// SameSite=None; Secure required for cross-site
```

### Lazy Loading Images

```html
<!-- Chrome native lazy loading -->
<img loading="lazy" src="image.jpg" alt="..." />

<!-- Intersection Observer fallback for other browsers -->
```

---

## 3. Firefox-Specific Issues

### CSS Scroll Snap

```css
/* Firefox scroll-snap may behave differently */
.container {
    scroll-snap-type: x mandatory;
    -webkit-overflow-scrolling: touch; /* For iOS, ignored in Firefox */
}
```

### Form Validation Styling

```css
/* Firefox form validation */
input:invalid {
    box-shadow: none; /* Remove Firefox default glow */
}

input:-moz-ui-invalid {
    box-shadow: none;
}
```

### Print Styles

```css
/* Firefox print differences */
@media print {
    .no-print {
        display: none !important;
    }
}
```

---

## 4. CSS Feature Detection

### @supports Usage

```css
/* Feature detection */
@supports (display: grid) {
    .container {
        display: grid;
    }
}

@supports not (gap: 1rem) {
    .flex-container > * + * {
        margin-left: 1rem;
    }
}

@supports (backdrop-filter: blur(10px)) or (-webkit-backdrop-filter: blur(10px)) {
    .glass {
        backdrop-filter: blur(10px);
        -webkit-backdrop-filter: blur(10px);
    }
}
```

### JavaScript Feature Detection

```typescript
// Feature detection pattern
const supportsIntersectionObserver = 'IntersectionObserver' in window;
const supportsResizeObserver = 'ResizeObserver' in window;
const supportsClipboard = navigator.clipboard !== undefined;

// Conditional usage
if (supportsIntersectionObserver) {
    const observer = new IntersectionObserver(callback);
} else {
    // Fallback or polyfill
}
```

---

## 5. Cross-Browser Test Checklist

### For Every PR/Feature:

- [ ] **Chrome** (latest) - Primary development
- [ ] **Safari** (macOS) - CSS quirks, date parsing
- [ ] **Safari** (iOS) - 100vh, touch events, video autoplay
- [ ] **Firefox** (latest) - Form styling, scroll behavior
- [ ] **Edge** (Chromium) - Usually same as Chrome

### Critical Checkpoints:

| Area | Safari | Chrome | Firefox |
|------|--------|--------|---------|
| Date parsing | ISO 8601 only | Flexible | Flexible |
| 100vh | dvh/fill-available | OK | OK |
| Flexbox gap | Missing in older versions | OK | OK |
| backdrop-filter | -webkit prefix | OK | OK |
| Smooth scroll | Use JS | OK | OK |
| Video autoplay | playsInline+muted | muted | muted |
| Clipboard API | May need fallback | OK | OK |

---

## 6. Senior Frontend Developer Competencies

### Cross-Browser Expertise

1. **Browser Engine Knowledge**
   - WebKit (Safari), Blink (Chrome/Edge), Gecko (Firefox)
   - Understanding rendering differences
   - DevTools proficiency (for each browser)

2. **Progressive Enhancement**
   - Core functionality must work in all browsers
   - Enhanced features in capable browsers
   - Graceful degradation patterns

3. **Feature Detection vs Browser Detection**
   ```typescript
   // ❌ Browser detection (fragile)
   if (navigator.userAgent.includes('Safari')) { }

   // ✅ Feature detection (robust)
   if ('IntersectionObserver' in window) { }
   ```

4. **Polyfill Strategy**
   - Knowledge of polyfills like core-js, whatwg-fetch
   - Understanding bundle size impact
   - Conditional loading (dynamic import)

### Performance Across Browsers

1. **Rendering Performance**
   - Preventing layout thrashing
   - Composite layer optimization
   - will-change usage

2. **Memory Management**
   - Memory leak detection
   - Garbage collection patterns
   - WeakMap/WeakSet usage

3. **Network Optimization**
   - Browser caching differences
   - Service Worker support
   - Prefetch/preload strategies

### Testing Mindset

1. **Automated Testing**
   - Playwright/Cypress multi-browser
   - BrowserStack/Sauce Labs integration
   - Visual regression testing

2. **Manual Testing Protocol**
   - Real device testing (especially iOS)
   - Testing across different viewports
   - Slow network simulation

3. **Bug Reproduction**
   - Browser version isolation
   - Minimal reproduction cases
   - Browser-specific debugging

### Accessibility Across Browsers

1. **Screen Reader Differences**
   - VoiceOver (Safari), NVDA (Firefox), JAWS (Chrome)
   - ARIA implementation differences
   - Focus management

2. **Keyboard Navigation**
   - Tab order consistency
   - Focus visible styles
   - Escape key handling

---

## 7. Browser-Specific Debugging

### Safari

```bash
# iOS Safari debugging
# On Mac: Safari > Develop > iPhone/iPad

# Watch for Safari-specific errors in Console
# "SecurityError", "NotAllowedError"
```

### Chrome

```bash
# DevTools > More tools > Rendering
# - Paint flashing
# - Layout shift regions
# - Layer borders
```

### Firefox

```bash
# DevTools > Inspector > Computed
# - Browser-specific CSS
# - Accessibility inspector
```

---

## 8. Utility Functions

```typescript
// utils/browser.ts

export const isSafari = () => {
    return /^((?!chrome|android).)*safari/i.test(navigator.userAgent);
};

export const isIOS = () => {
    return /iPad|iPhone|iPod/.test(navigator.userAgent);
};

export const isMobile = () => {
    return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
        navigator.userAgent
    );
};

// Feature detection helpers
export const supportsTouch = () => {
    return 'ontouchstart' in window || navigator.maxTouchPoints > 0;
};

export const supportsHover = () => {
    return window.matchMedia('(hover: hover)').matches;
};

export const prefersReducedMotion = () => {
    return window.matchMedia('(prefers-reduced-motion: reduce)').matches;
};
```

---

## 9. Common Tailwind Safari Fixes

```css
/* globals.css */

/* iOS Safari 100vh fix */
@supports (height: 100dvh) {
    .h-screen {
        height: 100dvh;
    }
}

/* Safari flexbox gap polyfill if needed */
@supports not (gap: 1rem) {
    .flex.gap-4 > * + * {
        margin-left: 1rem;
    }
}

/* Safari backdrop blur */
.backdrop-blur-sm {
    -webkit-backdrop-filter: blur(4px);
}
.backdrop-blur {
    -webkit-backdrop-filter: blur(8px);
}
.backdrop-blur-md {
    -webkit-backdrop-filter: blur(12px);
}
.backdrop-blur-lg {
    -webkit-backdrop-filter: blur(16px);
}
```

---

## Summary

| Priority | Action |
|----------|--------|
| 1 | Test Safari on every feature |
| 2 | Use ISO 8601 for date parsing |
| 3 | Use dvh or JS solution instead of 100vh |
| 4 | Use playsInline + muted for videos |
| 5 | Add -webkit prefix for backdrop-filter |
| 6 | Use passive listener for touch events |
| 7 | Minimum 16px font-size for inputs |
