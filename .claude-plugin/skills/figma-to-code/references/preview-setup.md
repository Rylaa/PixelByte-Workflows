# Preview Route Setup Guide

To run the visual validation script, you need to create a preview route in your project.

## Next.js App Router

**File:** `app/test-preview/page.tsx`

```tsx
'use client';

import { useSearchParams } from 'next/navigation';
import dynamic from 'next/dynamic';
import { Suspense } from 'react';

// Import components dynamically
const components: Record<string, React.ComponentType<any>> = {
  // Add new components here
  // 'HeroCard': dynamic(() => import('@/components/HeroCard')),
};

function PreviewContent() {
  const searchParams = useSearchParams();
  const componentName = searchParams.get('component');

  if (!componentName) {
    return <div>Component name required. Use ?component=ComponentName</div>;
  }

  const Component = components[componentName];

  if (!Component) {
    return <div>Component "{componentName}" not found in preview registry.</div>;
  }

  return (
    <div
      data-testid="preview-component"
      className="inline-block"
      style={{
        // Clean background for comparison
        background: 'white',
        // Auto to match Figma dimensions
        width: 'auto',
        height: 'auto'
      }}
    >
      <Component />
    </div>
  );
}

export default function TestPreviewPage() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <PreviewContent />
    </Suspense>
  );
}
```

## Next.js Pages Router

**File:** `pages/test-preview.tsx`

```tsx
import { useRouter } from 'next/router';
import dynamic from 'next/dynamic';

const components: Record<string, React.ComponentType<any>> = {
  // 'HeroCard': dynamic(() => import('@/components/HeroCard')),
};

export default function TestPreviewPage() {
  const router = useRouter();
  const { component: componentName } = router.query;

  if (!componentName || typeof componentName !== 'string') {
    return <div>Component name required</div>;
  }

  const Component = components[componentName];

  if (!Component) {
    return <div>Component not found</div>;
  }

  return (
    <div
      data-testid="preview-component"
      className="inline-block bg-white"
    >
      <Component />
    </div>
  );
}
```

## Vite / Create React App

**File:** `src/pages/TestPreview.tsx`

```tsx
import { useSearchParams } from 'react-router-dom';

// Import components
import { HeroCard } from '../components/HeroCard';

const components: Record<string, React.ComponentType<any>> = {
  'HeroCard': HeroCard,
};

export function TestPreview() {
  const [searchParams] = useSearchParams();
  const componentName = searchParams.get('component');

  if (!componentName) {
    return <div>Component name required</div>;
  }

  const Component = components[componentName];

  if (!Component) {
    return <div>Component not found</div>;
  }

  return (
    <div data-testid="preview-component" className="inline-block bg-white">
      <Component />
    </div>
  );
}
```

**Add to Router:** `src/App.tsx`

```tsx
import { TestPreview } from './pages/TestPreview';

// Inside Routes:
<Route path="/test-preview" element={<TestPreview />} />
```

## Adding New Components

Every new component must be added to the preview registry:

```tsx
const components: Record<string, React.ComponentType<any>> = {
  'Button': Button,
  'Card': Card,
  'HeroCard': HeroCard,  // Newly added
};
```

## Running Validation

### 1. Start Dev Server

```bash
npm run dev
```

### 2. Take Screenshot with Claude in Chrome MCP

```javascript
// 1. Get tab context (ALWAYS FIRST STEP!)
mcp__claude-in-chrome__tabs_context_mcp({
  createIfEmpty: true
})
// → Save the returned tabId

// 2. Navigate to preview page
mcp__claude-in-chrome__navigate({
  url: "http://localhost:3000/test-preview?component=HeroCard",
  tabId: <tab-id>
})

// 3. Wait for page to load
mcp__claude-in-chrome__computer({
  action: "wait",
  duration: 2,
  tabId: <tab-id>
})

// 4. Get accessibility tree (to find element ref)
mcp__claude-in-chrome__read_page({
  tabId: <tab-id>
})
// Read page output example:
// - main
//   - div "preview-container"
//     - article ref="ref_1" [data-testid="preview-component"]  ← Use this ref

// 5. Take screenshot
mcp__claude-in-chrome__computer({
  action: "screenshot",
  tabId: <tab-id>
})

// 6. (Optional) Scroll to element and zoom
mcp__claude-in-chrome__computer({
  action: "scroll_to",
  ref: "ref_1",
  tabId: <tab-id>
})

mcp__claude-in-chrome__computer({
  action: "zoom",
  region: [x0, y0, x1, y1],  // Preview component coordinates
  tabId: <tab-id>
})
```

**⚠️ IMPORTANT:** Target the same element as the Figma frame.

### 3. Compare with Claude Vision

Compare both visuals side by side:
- Figma screenshot (taken with Pixelbyte MCP)
- Browser element screenshot (just taken)

Claude Vision automatically detects differences.

## Outputs

After validation:
- Figma screenshot (returned as URL, provide to Claude Vision)
- `.qa/implementation.png` - Element screenshot
- If differences exist → List with TodoWrite → Fix → Re-test (max 3 iterations)

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Preview component not found" | Add `data-testid="preview-component"` |
| "Component not found" | Add to components registry |
| Size mismatch | Add `inline-block` and `bg-white` to container |
| Dev server connection error | Make sure `npm run dev` is running |
| Element ref not found | Check `read_page()` output or use `find()` |
| Tab ID error | Call `tabs_context_mcp()` first |
| Page not loaded | Wait with `computer({action: "wait"})` |
