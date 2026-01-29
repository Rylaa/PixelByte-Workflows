# Illustration Detection Reference

## Overview

This reference covers how to detect, classify, and handle illustrations and complex graphics extracted from Figma designs. Illustrations differ from icons in size, complexity, and export strategy.

## Illustration Detection Heuristics

### Size-Based Detection

Nodes exceeding icon dimensions are candidate illustrations:

| Dimension Check | Result |
|----------------|--------|
| width > 50px OR height > 50px | NOT an icon, likely illustration |
| width <= 64 AND height <= 64 | Icon (use icon template) |
| width > 64 OR height > 64 | Illustration (use illustration template) |

### Structural Heuristics

| Heuristic | Threshold | Classification |
|-----------|-----------|----------------|
| Vector group > 50px with >= 3 children | Large multi-part graphic | Illustration |
| >= 10 vector paths (`type="VECTOR"`) | Complex graphic | Illustration / Chart |
| Overlapping decorative layers with effects | Styled composition | Illustration |
| Composite layers (shadow + overlay) | Multi-layer graphic | Composite illustration |
| Frame nesting depth > 3 levels | Deeply nested structure | Likely illustration |

### Detection from Figma Node Data

```typescript
// Check vector count
const vectorCount = node.descendants.filter(d => d.type === "VECTOR").length;
if (vectorCount >= 10) {
  // Complex graphic - treat as illustration, export as PNG
}

// Check group size and children
if (node.type === "GROUP" || node.type === "FRAME") {
  const size = Math.max(node.width, node.height);
  const childCount = node.children?.length ?? 0;
  if (size > 50 && childCount >= 3) {
    // Large vector group - likely illustration
  }
}
```

## LLM Vision Analysis

### Trigger Conditions

Frames are flagged for LLM vision analysis based on these triggers:

| Trigger | Detection Method |
|---------|------------------|
| **Dark + Bright Siblings** | Frame has 2+ child frames where one has dark fills (luminosity < 0.27) and another has bright fills (luminosity > 0.5 AND saturation > 20%) |
| **Multiple Opacity Fills** | Frame children have identical hex color but 3+ different opacity values |
| **Gradient Overlay** | Vector child with gradient containing a stop with opacity >= 0.05 fading to a stop with opacity < 0.1 |
| **High Vector Count** | Frame contains > 10 descendants where `type="VECTOR"` |
| **Deep Nesting** | Frame nesting depth > 3 levels |

If a frame matches multiple triggers, each trigger is listed on a separate row.

**Luminosity and saturation formulas:**
```
luminosity = (R + G + B) / 3 / 255
  Dark:   luminosity < 0.27   (hex range #000000-#444444)
  Bright: luminosity > 0.5 AND saturation > 20%

saturation = (max(R,G,B) - min(R,G,B)) / 255
```

### Vision Analysis Workflow

1. Design Validator flags frames as potentially complex
2. Design Analyst passes them through without interpretation
3. Asset Manager makes the FINAL decision using LLM vision

**Process:**
```
1. Check Implementation Spec for "## Flagged for LLM Review" table
2. For each flagged frame:
   a. Take screenshot: figma_get_screenshot(file_key, [node_id], scale=2)
   b. Analyze screenshot using vision reasoning
   c. Record decision: DOWNLOAD_AS_IMAGE or GENERATE_AS_CODE
3. Update spec with decisions
4. Apply decision to download strategy
```

### Vision Analysis Reasoning Criteria

When analyzing a flagged frame screenshot, consider:

1. **Visual Complexity:**
   - Overlapping decorative layers? -> Likely illustration
   - Effects hard to replicate in code? -> Likely illustration
   - Stylized graphic rather than standard UI? -> Likely illustration

2. **Data Representation:**
   - Shows real/dynamic data that changes? -> Generate as code
   - Purely decorative/conceptual? -> Download as image

3. **Interactivity Potential:**
   - Users interact with individual parts? -> Generate as code
   - Static visual element? -> Download as image

4. **Code Complexity Estimate:**
   - Would coding require > 50 lines with complex positioning? -> Download as image
   - Achievable with simple shapes/gradients? -> Generate as code

### Decision Recording

Add decisions to the Implementation Spec:

```markdown
## Flagged Frames - LLM Decisions

| Node ID | Name | Decision | Reason |
|---------|------|----------|--------|
| 6:32 | GrowthSection | DOWNLOAD_AS_IMAGE | Decorative chart with shadow+color overlay effects, not real data |
```

| Decision | Action |
|----------|--------|
| DOWNLOAD_AS_IMAGE | Use `figma_get_screenshot` at 2x scale, save as PNG to `public/assets/images/` |
| GENERATE_AS_CODE | Skip in asset download, pass to code-generator agent |

## Flagged Frame Handling: Dark + Bright Siblings

### Composite Illustration Detection

Multi-layer illustrations may have shadow/background layers with dark fills that get exported instead of the visually dominant layer.

**Detection Process:**

1. When a frame has `exportSettings`, check children fills via `figma_get_node_details`
2. Classify each child by fill luminosity:
   - **Dark fills:** hex values #000000-#444444 range, `(R + G + B) / 3 < 68`
   - **Bright fills:** high saturation/luminosity, specific hue (not grayscale), luminosity > 50%
3. Identify layer roles:
   - `SHADOW_LAYER`: Frame with children having dark/grayscale fills
   - `PRIMARY_ILLUSTRATION`: Frame with children having bright/colored fills
   - `DECORATIVE_OVERLAY`: Vector with transparent gradient (opacity fading to 0)
4. Export decision:
   - If exportSettings node has DARK fills AND sibling has BRIGHT fills -> export the sibling with BRIGHT fills instead
   - If only one layer exists -> export as-is
   - If composite effect is intentional (shadow + color overlay) -> use `figma_get_screenshot` for the parent frame

**Example:**
```
Frame 6:32 (GrowthSection)
+-- 6:34 (children: #3c3c3c gradient) -> SHADOW_LAYER - skip
+-- 6:38 (children: #f2f20d fills)   -> PRIMARY_ILLUSTRATION - export this
+-- 6:44 (gradient white->transparent) -> DECORATIVE_OVERLAY - skip

Result: Export 6:38, not 6:34
```

## Semantic Naming Conventions

Use descriptive, semantic names for illustration assets. Never prefix illustrations with `icon-`.

| Asset Type | Naming Pattern | Examples |
|------------|---------------|----------|
| Charts | `chart-*.png` | `chart-growth.png`, `chart-revenue.png` |
| Illustrations | `illustration-*.png` | `illustration-hero.png`, `illustration-onboarding.png` |
| Graphs | `graph-*.png` | `graph-performance.png`, `graph-trends.png` |
| Icons | `icon-*.svg` | `icon-clock.svg`, `icon-settings.svg` |

**Rules:**
- Use kebab-case for all filenames
- Prefix complex vectors with descriptive name: `chart-growth.png`, `illustration-hero.png`
- Include size suffix for multiple resolutions: `hero-image-2x.png`
- Distinguish complex vectors from icons: use semantic names (`chart`, `graph`, `illustration`) NOT `icon-`

**Assets Inventory Example:**
```markdown
| Asset | Type | Node ID | Export Format |
|-------|------|---------|---------------|
| bar-chart | illustration | 6:34 | PNG |
| hero-illustration | illustration | 6:32 | PNG |
| icon-clock | icon | 1:890 | SVG |
```

## Export Format Decisions

| Content Type | Format | Rationale |
|-------------|--------|-----------|
| Simple illustrations (< 10 vector paths) | SVG | Scalable, small file size |
| Complex graphics (>= 10 vector paths) | PNG (2x scale) | Too many paths for clean SVG |
| Photographic / raster content | PNG / WebP | Raster content needs raster format |
| Charts with embedded text | PNG | Suppress text nodes, export as flat image |
| Icons (< 64px, simple paths) | SVG | Scalable, theme-able with rendering mode |

### Export Commands

**Simple illustrations (SVG):**
```
figma_export_assets:
  - file_key: {file_key}
  - node_ids: [{node_id}]
  - format: svg
  - scale: 1
```

**Complex illustrations (PNG):**
```
figma_export_assets:
  - file_key: {file_key}
  - node_ids: [{node_id}]
  - format: png
  - scale: 2
```

**Flagged composite illustrations (screenshot):**
```
figma_get_screenshot:
  - file_key: {file_key}
  - node_ids: [{node_id}]
  - scale: 2
  - format: png
```

### File Destinations

| Type | Path |
|------|------|
| SVG Icons | `public/assets/icons/{filename}.svg` |
| PNG Illustrations | `public/assets/images/{filename}.png` |
| WebP Images | `public/assets/images/{filename}.webp` |
| Bundled assets | `src/assets/{filename}.{ext}` |

## Platform-Specific Code Generation

### React / Next.js

```jsx
// Icon asset reference
<Image src="/assets/icons/icon-clock.svg" alt="Clock" width={32} height={32} />

// Illustration asset reference
<Image src="/assets/images/hero-illustration.png" alt="Hero" width={400} height={300} />
```

### SwiftUI

```swift
// Icon (small, <= 64px)
Image("icon-clock")
  .resizable()
  .renderingMode(.original)
  .frame(width: 32, height: 32)

// Illustration (larger images)
Image("growth-chart")
  .resizable()
  .aspectRatio(contentMode: .fit)
  .frame(width: 354, height: 132)

// Flagged illustration with clipping
Image("hero-illustration")
  .resizable()
  .aspectRatio(contentMode: .fit)
  .clipped()
  .cornerRadius(12)
```

## Cross-References

- `@references/image-with-text.md` - Text suppression for illustrations containing embedded text labels. When an illustration contains baked-in text (detected via `[contains-text]` annotation), suppress duplicate `Text()` generation and add `accessibilityLabel` instead.
- `@references/asset-classification-guide.md` - Icon vs illustration distinction criteria, asset type classification rules, and the full classification priority: `CHART_ILLUSTRATION > IMAGE_FILL > RASTER_IMAGE > COMPLEX_VECTOR > SIMPLE_ICON`.
