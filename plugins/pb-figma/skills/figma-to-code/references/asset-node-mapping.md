# Asset Node Mapping Reference

Cross-platform guide for constructing asset node maps from the design-analyst implementation spec and rendering images correctly per platform.

---

## Asset Node Map Construction Algorithm

The asset node map is built in three sequential steps before code generation begins.

### Step 1: Parse Asset Children from Implementation Spec

For each component in the "## Components" section, read the "Asset Children" property and parse each entry.

**Format:**

```
IMAGE:asset-name:NodeID:width:height
```

| Segment | Description | Example |
|---------|-------------|---------|
| `IMAGE` | Fixed prefix identifying an asset child | `IMAGE` |
| `asset-name` | Filename stem from the Assets Required table | `icon-clock` |
| `NodeID` | Figma node ID (`parent:child` format) | `3:230` |
| `width` | Asset width in pixels | `32` |
| `height` | Asset height in pixels | `32` |

**Parsing algorithm:**

```
For each component in "## Components" section:
  Read "Asset Children" property
  Parse format: IMAGE:asset-name:NodeID:width:height
  Add to assetNodeMap: { nodeId: { name, width, height } }
```

**Example assetNodeMap (JSON):**

```json
{
  "3:230": { "name": "icon-clock", "width": 32, "height": 32 },
  "6:32":  { "name": "hero-illustration", "width": 400, "height": 300 }
}
```

### Step 2: Cross-Reference with Downloaded Assets

Cross-reference the assetNodeMap entries with the "## Downloaded Assets" table to determine file type, local path, and rendering mode.

**React Downloaded Assets table format:**

| Asset | Local Path | Type | Next.js Optimized |
|-------|------------|------|-------------------|
| icon-clock | public/assets/icon-clock.svg | SVG | Yes - use next/image |
| hero-illustration | public/assets/hero.png | PNG | Yes - use next/image |

**SwiftUI Downloaded Assets table format:**

| Asset | Local Path | Fill Type | Template Compatible |
|-------|------------|-----------|---------------------|
| icon-clock | Assets.xcassets/icon-clock | #F2F20D | No - use .original |

For SwiftUI, add the rendering mode to the map:

```json
{
  "3:230": { "name": "icon-clock", "width": 32, "height": 32, "renderingMode": ".original" }
}
```

### Step 3: Use During Code Generation

When generating code for a component:

1. Check if the component contains any node IDs present in `assetNodeMap`
2. For asset nodes, **DO NOT** call `figma_generate_code`
3. Instead, generate the appropriate platform-specific image code manually

---

## MCP vs Manual Generation Decision Table

| Scenario | Approach | Reason |
|----------|----------|--------|
| Component with no assets | Use MCP `figma_generate_code` | MCP handles layout and styling |
| Asset node (icon/illustration) | Generate image code manually | MCP cannot access downloaded assets |
| Component containing assets | Use MCP for container, insert manual image code for assets | Hybrid approach |

---

## Rendering Mode Rules (SwiftUI)

| Downloaded Assets "Template Compatible" | SwiftUI Rendering Mode | Use Case |
|----------------------------------------|------------------------|----------|
| No - use .original | `.renderingMode(.original)` | Colored SVGs, multi-color icons |
| Yes - use .template | `.renderingMode(.template)` + `.foregroundColor()` | Tintable monochrome icons |
| Not specified | `.renderingMode(.original)` (safe default) | Fallback |

---

## Illustration vs Icon Detection

Determine the asset type from dimensions to select the correct code template:

| Dimension | Type | Template |
|-----------|------|----------|
| width <= 64 AND height <= 64 | ICON | Fixed `.frame(width:height:)` (SwiftUI) or fixed `width`/`height` attributes (React) |
| width > 64 OR height > 64 | ILLUSTRATION | `.aspectRatio(contentMode: .fit)` (SwiftUI) or `object-cover` (React) |

---

## Platform-Specific Image Syntax

### React (Next.js)

**Icons (small, <= 64px):**

```tsx
import Image from 'next/image';

<Image
  src="/assets/icon-clock.svg"
  alt="Clock icon"
  width={32}
  height={32}
  className="w-8 h-8"
/>
```

**Illustrations (larger images > 64px):**

```tsx
<Image
  src="/assets/hero.png"
  alt="Hero illustration"
  width={400}
  height={300}
  priority={isAboveFold}
  className="object-cover"
/>
```

### React (non-Next.js)

**Icons:**

```tsx
<img
  src="/assets/icon-clock.svg"
  alt="Clock icon"
  width={32}
  height={32}
  className="w-8 h-8"
/>
```

**Illustrations:**

```tsx
<img
  src="/assets/hero.png"
  alt="Hero illustration"
  loading="lazy"
  className="w-full h-auto object-cover"
/>
```

### React Icon Component Patterns

For SVG icons that need color control:

```tsx
// Option 1: lucide-react (when icon exists in library)
import { Clock } from 'lucide-react';
<Clock className="w-8 h-8 text-primary" />

// Option 2: Inline SVG component (unique design icons)
import ClockIcon from '@/assets/icons/clock.svg';
<ClockIcon className="w-8 h-8 fill-current text-primary" />
```

### SwiftUI

**Icons (small, typically < 64px):**

```swift
Image("icon-clock")
  .resizable()
  .renderingMode(.original)  // or .template
  .frame(width: 32, height: 32)
```

**Illustrations (larger images):**

```swift
Image("growth-chart")
  .resizable()
  .aspectRatio(contentMode: .fit)
  .frame(width: 354, height: 132)
```

**Flagged illustrations (from "Flagged for LLM Review" decided as DOWNLOAD_AS_IMAGE):**

```swift
Image("flagged-illustration")
  .resizable()
  .aspectRatio(contentMode: .fit)
  .frame(maxWidth: 354)
  .clipped()
  // Add .cornerRadius() if parent has border radius
```

---

## Design-Analyst Asset Children Marking

The design-analyst marks asset children when building the implementation spec.

**Process:**
1. For each component, check if any children match Node IDs in the "Assets Required" table
2. If a match is found, add to the "Asset Children" property using the `IMAGE:` format
3. Include dimensions from the Figma node

**Example spec output:**

| Property | Value |
|----------|-------|
| **Children** | IconFrame, TitleText, CheckmarkIcon |
| **Asset Children** | `IMAGE:icon-clock:3:230:32:32`, `IMAGE:growth-chart:6:32:354:132` |

Asset Children tells the code generator to use `Image("asset-name")` (SwiftUI) or `<img>`/`<Image>` (React) instead of generating code for that node via MCP.

---

## Common Mistakes

- Calling `figma_generate_code` on an asset node -- MCP cannot access downloaded assets, always generate image code manually
- Missing `renderingMode` on SwiftUI icons -- always check the Downloaded Assets table for Template Compatible status
- Using `.template` rendering mode on colored SVGs -- multi-color icons lose their colors; use `.original`
- Omitting `alt` text on React `<img>` and `<Image>` elements -- always provide descriptive alt text
- Using `next/image` for tiny icons under 64px -- overhead is unnecessary for small SVGs; prefer direct import or lucide-react
