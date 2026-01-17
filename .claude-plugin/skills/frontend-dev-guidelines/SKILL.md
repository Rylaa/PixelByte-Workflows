---
name: frontend-dev-guidelines
description: Frontend development guidelines for React/TypeScript applications. Modern patterns including Suspense, lazy loading, useSWR, file organization with features directory, shadcn/ui components, Tailwind CSS styling, Next.js App Router, performance optimization, and TypeScript best practices. Use when creating components, pages, features, fetching data, styling, routing, or working with frontend code.
---

# Frontend Dev Guidelines (Senior Level)

> **This skill is at senior frontend developer knowledge level.** Memorize patterns, recognize edge cases, reject anti-patterns.

---

## 1. Senior Developer Mindset

**Think performance-first:**
- Ask "is this causing unnecessary renders?" for every component
- Detect network waterfalls, use parallel fetch
- Monitor bundle size, lazy load heavy components

**Internalize patterns:**
- `React.FC<Props>` + TypeScript always
- `useSWR` with `suspense: true` is standard
- `SuspenseLoader` NEVER early return with spinner

**Reject anti-patterns:**
- Convert barrel file imports to direct imports
- Replace sequential await with `Promise.all()`
- Replace hardcoded colors with globals.css variables

---

## 2. Non-Negotiables (CRITICAL)

### Component Patterns

```typescript
// ✅ STANDARD PATTERN
import React from 'react';
import useSWR from 'swr';
import { SuspenseLoader } from '@/shared/components/SuspenseLoader';

const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

interface MyComponentProps {
    id: string;
    className?: string;
}

export const MyComponent: React.FC<MyComponentProps> = ({ id, className }) => {
    const { data } = useSWR(`key-${id}`, () => api.get(id), { suspense: true });
    return <div className={cn("base-class", className)}>{data?.title}</div>;
};

// Usage - with Suspense boundary
<SuspenseLoader>
    <MyComponent id="123" />
</SuspenseLoader>
```

### Performance Rules (Top 15)

| Priority | Rule | Description |
|----------|------|-------------|
| CRITICAL | `Promise.all()` | Parallelize independent async operations |
| CRITICAL | Direct imports | Avoid barrel files, import directly |
| CRITICAL | `next/dynamic` | Dynamic import heavy components |
| CRITICAL | Suspense boundaries | Stream content, prevent waterfalls |
| HIGH | Defer await | Move await to the branch where it's used |
| HIGH | Minimize RSC serialization | Send less data to client |
| HIGH | `React.cache()` | Per-request deduplication |
| MEDIUM | SWR deduplication | Auto request deduplication |
| MEDIUM | Functional setState | `setCount(c => c + 1)` stable callback |
| MEDIUM | Lazy state init | `useState(() => expensiveCalc())` |
| MEDIUM | `startTransition` | For non-urgent updates |
| MEDIUM | `content-visibility` | CSS optimize for long lists |
| LOW | Index maps | Use Map for repeated lookups |
| LOW | `toSorted()` | For immutable sort |
| LOW | Set/Map lookups | O(1) lookup instead of array |

### Accessibility Rules (Top 10)

| Rule | Description |
|------|-------------|
| `aria-label` on icon buttons | ALWAYS add aria-label to icon-only buttons |
| `button` vs `a/Link` | Action = button, Navigation = a/Link |
| `alt` on images | Add alt to every img (decorative: `alt=""`) |
| Keyboard handlers | Add keyboard support to interactive elements |
| Heading hierarchy | Maintain h1 → h2 → h3 order |
| `autocomplete` on inputs | Add autocomplete to form inputs |
| Inline errors | Show errors next to field |
| Submit button enabled | Don't disable submit, show loading spinner |
| Virtualize long lists | Virtualize lists with 50+ items |
| `prefers-reduced-motion` | Check motion preference in animations |

---

## 3. Browser Compatibility (SAFARI CRITICAL)

> **Check Safari on every code change!** Safari is the most problematic browser.

| Issue | Safari Fix | Reference |
|-------|------------|-----------|
| `new Date('2024-01-15 10:30:00')` | ISO 8601: `'2024-01-15T10:30:00'` | browser-compatibility.md |
| `height: 100vh` | `height: 100dvh` or `-webkit-fill-available` | browser-compatibility.md |
| `backdrop-filter` | Add `-webkit-backdrop-filter` prefix | browser-compatibility.md |
| Video autoplay | `playsInline` + `muted` attributes | browser-compatibility.md |
| Input zoom (iOS) | `font-size: 16px` minimum | browser-compatibility.md |
| Touch events | `{ passive: true }` listener | browser-compatibility.md |
| Clipboard API | Fallback textarea method | browser-compatibility.md |

### Test Checklist (Every PR):
- [ ] Chrome (latest)
- [ ] Safari (macOS)
- [ ] Safari (iOS) - Real device or simulator
- [ ] Firefox (latest)

---

## 4. Anti-Patterns (NEVER DO)

| Anti-Pattern | Correct Pattern | Reference |
|--------------|-----------------|-----------|
| `if (isLoading) return <Spinner />` | Wrap with `<SuspenseLoader>` | core-patterns.md |
| Sequential await | Use `Promise.all()` | performance-guide.md |
| Barrel file import | Use direct import | performance-guide.md |
| Hardcoded color `bg-[#fe4601]` | CSS variable `bg-orange-1` | styling-routing.md |
| `<div onClick>` | `<button onClick>` | accessibility-guide.md |
| Icon button without label | Add `aria-label` | accessibility-guide.md |
| Server data in useState | Use `useSWR` | core-patterns.md |
| `next/router` import | Use `next/navigation` | styling-routing.md |
| `dangerouslySetInnerHTML` | DOMPurify sanitization | security-error-handling.md |
| `any` type | `unknown` + type guard | advanced-typescript.md |
| Type assertion `as User` | Runtime validation (Zod) | advanced-typescript.md |

---

## 5. Decision Trees

### useState vs Zustand vs SWR

```
What's the data source?
├─ Server/API → SWR (suspense: true)
├─ Form input → useState (local)
├─ UI state (modal, sidebar) → useState (local)
└─ Shared across components?
   ├─ Server data → SWR (auto-shares)
   └─ Client state → Zustand (ONLY)
```

### Server vs Client Component

```
What does the component do?
├─ Static content → Server Component
├─ Data fetch (no interaction) → Server Component
├─ Needs useState/useEffect → Client Component
├─ Has onClick/onChange handler → Client Component
├─ Uses Browser API → Client Component
└─ Third-party client library → Client Component
```

### useMemo vs React.memo vs useCallback

```
What are you optimizing?
├─ Expensive calculation → useMemo
├─ Object/array reference stability → useMemo
├─ Function reference stability → useCallback
├─ Child component re-render → React.memo (on child)
└─ Event handler in dependency → useCallback
```

### Lazy Loading Decision

```
Is the component heavy?
├─ DataGrid/Table → lazy load
├─ Chart/Graph → lazy load
├─ Rich text editor → lazy load
├─ Video player → lazy load
├─ Small utility component → don't lazy load
└─ Above-the-fold critical → don't lazy load
```

---

## 6. Quick Reference

> **For detailed info:** [styling-routing.md](resources/styling-routing.md)

| Topic | Source |
|-------|--------|
| Import aliases (`@/`, `@/shared`, `@/features`) | styling-routing.md |
| Feature directory structure | styling-routing.md |
| Route groups (`(dashboard)`, `(main)`, etc.) | styling-routing.md |
| Color palettes (globals.css) | styling-routing.md |

---

## 7. Resources (Detailed Info)

| Topic | Resource |
|-------|----------|
| Component, data fetching, React 19 hooks, state | [core-patterns.md](resources/core-patterns.md) |
| All performance rules (45 rules) | [performance-guide.md](resources/performance-guide.md) |
| Accessibility patterns (WCAG 2.1) | [accessibility-guide.md](resources/accessibility-guide.md) |
| Tailwind, routing, file organization | [styling-routing.md](resources/styling-routing.md) |
| Testing patterns, complete examples | [testing-examples.md](resources/testing-examples.md) |
| Safari/Chrome/Firefox compatibility, Senior FE skills | [browser-compatibility.md](resources/browser-compatibility.md) |
| **XSS/CSRF prevention, error boundaries, resilience** | [security-error-handling.md](resources/security-error-handling.md) |
| **Generics, type guards, utility types, inference** | [advanced-typescript.md](resources/advanced-typescript.md) |

---

## Quick Template

```typescript
import React from 'react';
import useSWR from 'swr';
import { Card, CardHeader, CardTitle, CardContent } from '@/shared/components/ui/card';
import { cn } from '@/shared/utils/cn';
import { featureApi } from '../api/featureApi';

interface FeatureCardProps {
    id: string;
    className?: string;
}

export const FeatureCard: React.FC<FeatureCardProps> = ({ id, className }) => {
    const { data } = useSWR(`feature-${id}`, () => featureApi.get(id), { suspense: true });

    return (
        <Card className={cn("w-full", className)}>
            <CardHeader>
                <CardTitle>{data?.title}</CardTitle>
            </CardHeader>
            <CardContent>
                <p className="text-muted-foreground">{data?.description}</p>
            </CardContent>
        </Card>
    );
};
```

---

## Related Skills

- **react-best-practices**: Full 45 performance rules (Vercel Engineering)
- **web-design-guidelines**: Full UI/UX/a11y rules (Vercel Labs)
- **error-tracking**: Sentry integration
