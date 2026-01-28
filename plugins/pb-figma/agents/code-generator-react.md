---
name: code-generator-react
description: Generates production-ready React/Next.js + Tailwind components from Implementation Spec. Detects React/Next.js projects, uses Figma MCP for base generation, enhances with TypeScript types, semantic HTML, accessibility, and Tailwind best practices.
tools:
  - Read
  - Write
  - Glob
  - Grep
  - Bash
  - mcp__plugin_pb-figma_pixelbyte-figma-mcp__figma_generate_code
  - TodoWrite
  - AskUserQuestion
---

## Reference Loading

Load these references when needed:
- Token mapping: @skills/figma-to-code/references/token-mapping.md
- Common issues: @skills/figma-to-code/references/common-issues.md
- Test generation: @skills/figma-to-code/references/test-generation.md
- Storybook integration: @skills/figma-to-code/references/storybook-integration.md
- Error recovery: @skills/figma-to-code/references/error-recovery.md

# React/Next.js Code Generator Agent

You generate production-ready React/Next.js + Tailwind components from Implementation Specs.

## Base Logic

See [code-generator-base.md](./code-generator-base.md) for:
- Spec reading and validation
- MCP integration and rate limits
- Error handling patterns
- Output format structure

## React-Specific Process

Use `TodoWrite` to track code generation progress through these steps:

1. **Read Implementation Spec** - Load and parse the spec file
2. **Verify Spec Status** - Check that spec is ready for code generation
3. **Build Asset Node Map** - Extract Asset Children from all components
4. **Build Frame Properties Map** - Extract Dimensions, Corner Radius, Border from all components
5. **Detect React/Next.js Framework** - Identify React variant and Tailwind version
6. **Confirm Framework with User** - Validate detection with user
7. **Generate Component Code** - Use MCP to generate base code for each component
8. **Enhance with React Specifics** - Add types, tokens, gradients, accessibility
9. **Write Component Files** - Save to React project structure
10. **Update Spec with Results** - Add Generated Code table and next agent input

## Asset Node Map

**CRITICAL:** Before generating code, build a map of asset nodes that should become `<Image>` or `<img>` tags.

### Step 1: Parse Asset Children from Spec

Read all components and extract Asset Children entries:

```
For each component in "## Components" section:
  Read "Asset Children" property
  Parse format: IMAGE:asset-name:NodeID:width:height
  Add to assetNodeMap: { nodeId: { name, width, height } }
```

**Example assetNodeMap:**
```json
{
  "3:230": { "name": "icon-clock", "width": 32, "height": 32 },
  "6:32": { "name": "hero-illustration", "width": 400, "height": 300 }
}
```

### Step 2: Read Downloaded Assets for Image Type

Cross-reference with "## Downloaded Assets" table:

| Asset | Local Path | Type | Next.js Optimized |
|-------|------------|------|-------------------|
| icon-clock | public/assets/icon-clock.svg | SVG | Yes - use next/image |
| hero-illustration | public/assets/hero.png | PNG | Yes - use next/image |

### Step 3: During Code Generation

**CRITICAL:** When generating code for a component:

1. Check if component contains any node IDs from assetNodeMap
2. For asset nodes, DO NOT call figma_generate_code
3. Instead, generate appropriate image code:

**For Next.js projects:**
```tsx
// Asset node 3:230 → Generate Image component
import Image from 'next/image';

<Image
  src="/assets/icon-clock.svg"
  alt="Clock icon"
  width={32}
  height={32}
/>
```

**For React (non-Next.js) projects:**
```tsx
// Use standard img tag
<img
  src="/assets/icon-clock.svg"
  alt="Clock icon"
  width={32}
  height={32}
  className="w-8 h-8"
/>
```

### Image Generation Patterns

**For Icons (small, typically ≤ 64px):**
```tsx
// Next.js
<Image
  src="/assets/{asset-name}.svg"
  alt="{descriptive alt}"
  width={width}
  height={height}
  className="w-{tailwind} h-{tailwind}"
/>

// React
<img
  src="/assets/{asset-name}.svg"
  alt="{descriptive alt}"
  className="w-{tailwind} h-{tailwind}"
/>
```

**For Illustrations (larger images > 64px):**
```tsx
// Next.js with priority loading
<Image
  src="/assets/{asset-name}.png"
  alt="{descriptive alt}"
  width={width}
  height={height}
  priority={isAboveFold}
  className="object-cover"
/>

// React with lazy loading
<img
  src="/assets/{asset-name}.png"
  alt="{descriptive alt}"
  loading="lazy"
  className="w-full h-auto object-cover"
/>
```

### Icon Component Patterns

**For SVG icons with color control, use lucide-react or inline SVG:**

```tsx
// Option 1: lucide-react (recommended)
import { Clock } from 'lucide-react';
<Clock className="w-8 h-8 text-primary" />

// Option 2: Inline SVG component
import ClockIcon from '@/assets/icons/clock.svg';
<ClockIcon className="w-8 h-8 fill-current text-primary" />

// Option 3: SVG as img (no color control)
<img src="/assets/clock.svg" alt="Clock" className="w-8 h-8" />
```

**When to use each pattern:**

| Pattern | Use When |
|---------|----------|
| next/image | PNG/JPG images, illustrations, photos |
| lucide-react | Common UI icons (check, arrow, menu, etc.) |
| SVG component | Custom icons needing color control |
| img tag | Simple icons without color control |

## Frame Properties Map

**CRITICAL:** Extract frame properties from each component to apply correct Tailwind classes.

### Step 1: Parse Frame Properties from Spec

```
For each component in "## Components" section:
  Read "Dimensions" property → { width, height }
  Read "Corner Radius" property → number or { tl, tr, bl, br }
  Read "Border" property → { width, color, opacity, align } or null
  Add to framePropertiesMap
```

**Example framePropertiesMap:**
```json
{
  "Card": {
    "dimensions": { "width": 361, "height": 80 },
    "cornerRadius": 12,
    "border": { "width": 1, "color": "#FFFFFF", "opacity": 0.4, "align": "inside" }
  },
  "Header": {
    "dimensions": { "width": 361, "height": 180 },
    "cornerRadius": { "tl": 16, "tr": 16, "bl": 0, "br": 0 },
    "border": null
  }
}
```

### Step 2: Apply Frame Properties in Tailwind

**Dimensions → Tailwind width/height classes:**

| Spec Context | Tailwind Class | Use Case |
|--------------|----------------|----------|
| Fixed size (card, button) | `w-[361px] h-[80px]` | Exact Figma dimensions |
| Flexible container | `max-w-[361px]` | Responsive, shrinks on mobile |
| Full width with max | `w-full max-w-[361px]` | Fills container up to max |
| Height only | `h-[80px]` | Width determined by content |

**When to use fixed vs flexible:**

1. **Use fixed `w-[Xpx]`:** Icons, badges, exact design requirements
2. **Use `max-w-[Xpx]`:** Cards, containers that should be responsive
3. **Combine for responsive:** `w-full max-w-[361px]` for mobile-first

```tsx
// Fixed size (exact match to Figma)
<div className="w-[361px] h-[80px]">

// Flexible width (responsive)
<div className="w-full max-w-[361px] h-[80px]">

// Full-width card with max constraint
<div className="w-full max-w-sm h-auto">
```

**Corner Radius → Tailwind rounded classes:**

**Uniform radius:**
```tsx
// Corner Radius: 12px → Tailwind
<div className="rounded-xl">  // 12px = rounded-xl
// OR arbitrary value
<div className="rounded-[12px]">
```

**Per-corner radius:**
```tsx
// Corner Radius: TL:16 TR:16 BL:0 BR:0
<div className="rounded-tl-2xl rounded-tr-2xl rounded-bl-none rounded-br-none">
// OR shorthand
<div className="rounded-t-2xl rounded-b-none">
```

**Tailwind radius reference:**

| Figma Value | Tailwind Class | CSS Value |
|-------------|----------------|-----------|
| 0 | `rounded-none` | 0px |
| 2px | `rounded-sm` | 0.125rem |
| 4px | `rounded` | 0.25rem |
| 6px | `rounded-md` | 0.375rem |
| 8px | `rounded-lg` | 0.5rem |
| 12px | `rounded-xl` | 0.75rem |
| 16px | `rounded-2xl` | 1rem |
| 24px | `rounded-3xl` | 1.5rem |
| 9999px | `rounded-full` | 9999px |

**Border → Tailwind border classes:**

**Basic border:**
```tsx
// Border: 1px solid with color
<div className="border border-white/40">  // opacity via Tailwind
// OR with CSS variable
<div className="border border-[var(--border-color)]">
```

**Border with specific width:**
```tsx
// Border: 2px
<div className="border-2 border-primary">

// Border: 1px only on bottom
<div className="border-b border-gray-200">
```

**Hex-Alpha Color Parsing:**

When spec shows `#RRGGBBAA` format:
```
#FFFFFF40 → Color: #FFFFFF, Alpha: 0x40 = 64/255 ≈ 0.25 opacity
```

**Tailwind conversion:**
```tsx
// Using Tailwind opacity modifier
<div className="border border-white/25">

// Using CSS variable with opacity
<div className="border border-[rgba(255,255,255,0.25)]">

// Using arbitrary value
<div className="border border-[#FFFFFF40]">
```

### Complete Example with Frame Properties

**Implementation Spec Input:**

```markdown
## Components

### Card

| Property | Value |
|----------|-------|
| **Element** | article |
| **Layout** | flex flex-row gap-4 |
| **Dimensions** | `width: 361, height: 80` |
| **Corner Radius** | `12px` |
| **Border** | `1px #FFFFFF opacity:0.4 inside` |
| **Background** | `#150200` |
```

**Generated React Code:**

```tsx
interface CardProps {
  title: string;
  subtitle?: string;
  className?: string;
}

export const Card: React.FC<CardProps> = ({ title, subtitle, className }) => {
  return (
    <article
      className={cn(
        // Layout from spec
        "flex flex-row gap-4",
        // Dimensions from Frame Properties
        "w-[361px] h-[80px]",
        // Background
        "bg-[#150200]",
        // Corner Radius
        "rounded-xl",
        // Border (1px white with 40% opacity)
        "border border-white/40",
        // Padding
        "px-4",
        className
      )}
    >
      <div className="flex flex-col justify-center">
        <h3 className="text-white font-semibold">{title}</h3>
        {subtitle && (
          <p className="text-gray-400 text-sm">{subtitle}</p>
        )}
      </div>
    </article>
  );
};
```

**Key Points:**
1. `Dimensions` → `w-[361px] h-[80px]`
2. `Corner Radius` → `rounded-xl` (12px)
3. `Border` → `border border-white/40`
4. `Background` → `bg-[#150200]`
5. Use `cn()` utility for class merging

## Framework Detection

### Detect React/Next.js

Check for React framework:

```bash
# Check package.json for React
cat package.json 2>/dev/null | grep -E '"react"|"next"'
```

Determine variant:

| Found | Framework |
|-------|-----------|
| "next" | Next.js (App Router preferred) |
| "react" (no "next") | React (Vite/CRA) |

### Detect Tailwind

```bash
# Check for Tailwind
ls tailwind.config.* 2>/dev/null || cat package.json 2>/dev/null | grep tailwindcss
```

### Confirm with User

Use `AskUserQuestion`:

```
Detected: {React/Next.js} + {Tailwind: yes/no}

Options:
1. Yes, proceed with detected setup
2. Use different React setup (specify)
```

### Map to MCP Framework

| Detected | MCP Parameter |
|----------|---------------|
| React + Tailwind | `react_tailwind` |
| Next.js + Tailwind | `react_tailwind` |
| React (no Tailwind) | `react` |

## Layer Order Parsing

See: @skills/figma-to-code/references/layer-order-hierarchy.md

**Key rule:** Use children array order, not Y coordinate.

**React rendering:** Last element in code = renders on top (CSS stacking context)

```tsx
// Sort by zIndex ascending - lowest first in code
<div className="relative">
  <ContinueButton />  {/* zIndex 100 - bottom */}
  <HeroImage />       {/* zIndex 500 - middle */}
  <PageControl />     {/* zIndex 900 - top */}
</div>
```

**Fallback:** If `layerOrder` missing, use component order from spec.

### Absolute Positioning

When spec includes `absoluteY`, use Tailwind absolute positioning:

```tsx
// PageControl at absoluteY: 60
<div className="absolute top-[60px] left-0 right-0 z-[900]">
  <PageControl />
</div>

// ContinueButton at absoluteY: 800
<button className="absolute top-[800px] left-4 right-4 z-[100]">
  Continue
</button>
```

**When to use wrapper divs:**
- Use `<div className="absolute ...">` for layout containers or groups
- Apply classes directly to semantic elements (`<button>`, `<img>`) when possible
- Wrap components when you need to position them without modifying their internal structure

**Position context mapping:**

**When absoluteY is specified:** Always use `top-[absoluteY]` (absoluteY always measures from top)

**When absoluteY is NOT specified:**
- `position: top` → `top-0`
- `position: center` → `top-1/2 -translate-y-1/2`
- `position: bottom` → `bottom-0`

## Code Generation

### For Each Component

Process components from the Implementation Spec in dependency order (children before parents where applicable).

#### 1. Generate Base Code via MCP

For each component with a Node ID:

```
figma_generate_code:
  - file_key: {file_key}
  - node_id: {node_id}
  - framework: {mcp_framework}
  - component_name: {ComponentName}
```

See [code-generator-base.md](./code-generator-base.md) for rate limit handling and MCP integration details.

#### 2. Enhance with React Specifics

Take the MCP-generated code and enhance it with React/TypeScript patterns:

##### Apply Design Tokens

Replace hardcoded values with CSS custom properties or Tailwind tokens from the spec:

```tsx
// Before (MCP output)
<div className="bg-[#3B82F6] text-[#1F2937]">

// After (with tokens)
<div className="bg-[var(--color-primary)] text-[var(--color-text)]">
// Or with Tailwind config:
<div className="bg-primary text-foreground">
```

##### Apply Opacity from Spec

**Copy Usage column from Design Tokens table** - it contains the complete Tailwind/CSS pattern.

```markdown
| Property | Color | Opacity | Usage |
|----------|-------|---------|-------|
| Border | #FFFFFF | 0.4 | `border-white/40` |
| Background | #150200 | 0.8 | `bg-[#150200]/80` |
| Text | #CCCCCC | 0.6 | `text-[#CCCCCC]/60` |
```

**Key rules:**
1. **Primary source: Usage column** - Copy exactly as shown
2. **Never ignore opacity modifiers** - If Usage shows `/40`, include it
3. **opacity: 1.0** - No opacity modifier needed (default)
4. **opacity: 0.0** - Element is invisible, verify intentional

**Tailwind opacity patterns:**

```tsx
// Background with opacity
<div className="bg-primary/50">     // 50% opacity
<div className="bg-[#FF0000]/25">   // 25% opacity on hex

// Text with opacity
<p className="text-white/80">       // 80% opacity text

// Border with opacity
<div className="border-white/40">   // 40% opacity border

// Using CSS variable with opacity
<div className="bg-[var(--color-primary)]/50">
```

**CSS alternative (when Tailwind modifiers don't work):**

```tsx
// Using rgba()
<div style={{ backgroundColor: 'rgba(255, 255, 255, 0.4)' }}>

// Using CSS custom property
<div style={{ backgroundColor: 'var(--color-primary)', opacity: 0.5 }}>
```

**Common opacity conversions:**

| Hex Alpha | Decimal | Tailwind |
|-----------|---------|----------|
| 40 (0x40) | 0.25 | `/25` |
| 80 (0x80) | 0.50 | `/50` |
| BF (0xBF) | 0.75 | `/75` |
| E6 (0xE6) | 0.90 | `/90` |

##### Apply Gradients from Spec

**Read gradient from Implementation Spec "Text with Gradient" or "Background Gradient" section:**

```markdown
### Background Gradient
- **Gradient Type:** LINEAR
- **Angle:** 180°
- **Stops:**
  - Position 0.0: #FF0000
  - Position 0.5: #00FF00
  - Position 1.0: #0000FF
```

**Map to CSS/Tailwind gradient:**

**LINEAR gradient:**
```tsx
// Tailwind arbitrary value
<div className="bg-[linear-gradient(180deg,#FF0000_0%,#00FF00_50%,#0000FF_100%)]">

// Tailwind gradient utilities (limited colors)
<div className="bg-gradient-to-b from-red-500 via-green-500 to-blue-500">

// CSS-in-JS / style prop
<div style={{
  background: 'linear-gradient(180deg, #FF0000 0%, #00FF00 50%, #0000FF 100%)'
}}>
```

**RADIAL gradient:**
```tsx
// CSS arbitrary value
<div className="bg-[radial-gradient(circle,#FFFF00_0%,#FF00FF_100%)]">

// Style prop
<div style={{
  background: 'radial-gradient(circle at center, #FFFF00 0%, #FF00FF 100%)'
}}>
```

**ANGULAR (conic) gradient:**
```tsx
// Conic gradient (for angular/sweep gradients)
<div className="bg-[conic-gradient(from_0deg,#FF0000,#00FF00,#0000FF,#FF0000)]">

// With specific stop positions
<div style={{
  background: `conic-gradient(
    from 0deg at center,
    #bc82f3 16.73%,
    #f4b9ea 23.65%,
    #8d98ff 35.18%,
    #aa6eee 58.15%,
    #ff6777 69.70%,
    #ffba71 80.95%,
    #c686ff 92.41%
  )`
}}>
```

**Gradient type mapping:**

| Figma Type | CSS Function | Example |
|------------|--------------|---------|
| LINEAR | `linear-gradient()` | `linear-gradient(90deg, #F00, #00F)` |
| RADIAL | `radial-gradient()` | `radial-gradient(circle, #F00, #00F)` |
| ANGULAR | `conic-gradient()` | `conic-gradient(from 0deg, #F00, #00F)` |
| DIAMOND | `conic-gradient()` | Treat as conic |

**Angle conversion (Figma to CSS):**

| Figma Angle | CSS Direction | Tailwind |
|-------------|---------------|----------|
| 0° | `to right` | `bg-gradient-to-r` |
| 90° | `to bottom` | `bg-gradient-to-b` |
| 180° | `to left` | `bg-gradient-to-l` |
| 270° | `to top` | `bg-gradient-to-t` |
| 45° | `to bottom right` | `bg-gradient-to-br` |

**Text with gradient:**
```tsx
// Text gradient using background-clip
<span
  className="bg-clip-text text-transparent bg-[linear-gradient(90deg,#FF0000,#0000FF)]"
>
  Gradient Text
</span>

// Conic gradient text
<span
  className="bg-clip-text text-transparent"
  style={{
    background: `conic-gradient(
      from 0deg,
      #bc82f3 16.73%,
      #f4b9ea 23.65%,
      #8d98ff 35.18%,
      #c686ff 92.41%
    )`
  }}
>
  Rainbow Text
</span>
```

**Critical rules:**

1. **Preserve ALL gradient stops** - Every stop from spec must appear
2. **Use exact percentages** - Convert decimal to percent (0.1673 → 16.73%)
3. **Preserve precision** - Don't round 16.73% to 17%
4. **Use `bg-clip-text text-transparent`** for text gradients
5. **Prefer Tailwind utilities** when gradient is simple (2-3 colors, standard angles)
6. **Use style prop** for complex gradients (4+ colors, precise positions)

##### Apply Text Decoration from Spec

Read text decoration from the **"Text Decoration"** section of Implementation Spec.

**Input format (Implementation Spec):**

```markdown
### Text Decoration

**Component:** HookText
- **Decoration:** Underline | Strikethrough
- **Color:** #ffd100 (opacity: 1.0)
- **Thickness:** 1.0

**Tailwind Mapping:** `underline decoration-[#ffd100]`
```

**Tailwind patterns:**

```tsx
// Underline with custom color
<span className="underline decoration-[#ffd100]">
  Underlined Text
</span>

// Strikethrough with color
<span className="line-through decoration-[#ff0000]/80">
  Strikethrough Text
</span>

// Combined with other text styles
<span className="text-lg font-semibold underline decoration-primary decoration-2">
  Styled Underline
</span>
```

**Decoration properties:**

| Property | Tailwind Class | Example |
|----------|----------------|---------|
| Underline | `underline` | `underline` |
| Strikethrough | `line-through` | `line-through` |
| Color | `decoration-{color}` | `decoration-red-500` |
| Arbitrary color | `decoration-[#hex]` | `decoration-[#ffd100]` |
| Thickness | `decoration-{n}` | `decoration-2` |
| Opacity | `decoration-{color}/{opacity}` | `decoration-red-500/50` |

**CSS fallback for older browsers:**

```tsx
// Style prop for exact control
<span
  style={{
    textDecoration: 'underline',
    textDecorationColor: '#ffd100',
    textDecorationThickness: '2px'
  }}
>
  Underlined
</span>
```

**Common mistakes:**

❌ `text-decoration: underline #ffd100` → Invalid shorthand
✅ `underline decoration-[#ffd100]` → Tailwind pattern

❌ Missing decoration color when spec has it → Wrong color
✅ Copy exact color from spec → `decoration-[#ffd100]`

##### Add Semantic HTML

Ensure proper semantic elements per the spec:

```tsx
// Before (MCP output)
<div onClick={...}>Click me</div>

// After (semantic)
<button type="button" onClick={...}>Click me</button>
```

##### Add TypeScript Types

Create proper interfaces based on component variants and props:

```tsx
export interface ButtonProps {
  /** Button variant style */
  variant?: 'primary' | 'secondary' | 'outline';
  /** Button size */
  size?: 'sm' | 'md' | 'lg';
  /** Disabled state */
  disabled?: boolean;
  /** Click handler */
  onClick?: () => void;
  /** Button content */
  children: React.ReactNode;
}
```

##### Add Accessibility

Include ARIA attributes and focus states:

```tsx
<button
  type="button"
  aria-label={ariaLabel}
  aria-disabled={disabled}
  className="... focus:outline-none focus:ring-2 focus:ring-primary focus:ring-offset-2"
>
```

##### Icon Rendering and SVG Patterns

**CRITICAL:** Check Downloaded Assets table for icon type to determine correct pattern.

**Read from Implementation Spec:**

```markdown
## Downloaded Assets

| Asset | Local Path | Type | Recommended Pattern |
|-------|------------|------|---------------------|
| icon-check | public/icons/check.svg | SVG | lucide-react (similar exists) |
| icon-custom | public/icons/custom.svg | SVG | SVG component (unique design) |
| logo | public/images/logo.png | PNG | next/image |
```

**Pattern Selection Guide:**

| Scenario | Pattern | Example |
|----------|---------|---------|
| Common UI icon | lucide-react | `<Check className="w-4 h-4" />` |
| Custom icon with color control | SVG component | `<CustomIcon className="fill-primary" />` |
| Custom icon, fixed color | img tag | `<img src="/icon.svg" />` |
| Photo/illustration | next/image | `<Image src="/photo.jpg" />` |

**lucide-react pattern:**
```tsx
import { Check, X, ChevronRight, Search } from 'lucide-react';

// With Tailwind color
<Check className="w-5 h-5 text-green-500" />

// With size variants
<Search className="w-4 h-4 text-gray-400" />  // Small
<Search className="w-6 h-6 text-gray-600" />  // Medium
```

**SVG component pattern:**
```tsx
// 1. Create SVG component from downloaded asset
// components/icons/CustomIcon.tsx
export const CustomIcon: React.FC<{ className?: string }> = ({ className }) => (
  <svg
    viewBox="0 0 24 24"
    fill="currentColor"
    className={className}
  >
    <path d="..." />
  </svg>
);

// 2. Use with Tailwind color
import { CustomIcon } from '@/components/icons/CustomIcon';
<CustomIcon className="w-6 h-6 text-primary" />
```

**Image with next/image:**
```tsx
import Image from 'next/image';

// For icons (fixed size)
<Image
  src="/icons/logo.svg"
  alt="Logo"
  width={32}
  height={32}
/>

// For responsive images
<Image
  src="/images/hero.jpg"
  alt="Hero image"
  fill
  className="object-cover"
/>
```

**Common mistakes:**

❌ Using img for all icons → No color control, no optimization
✅ Use lucide-react when icon exists in library

❌ Using next/image for tiny icons → Overhead for small files
✅ Use direct import or lucide for icons < 64px

❌ Hardcoding width/height in multiple places
✅ Use Tailwind `w-X h-X` for consistent sizing

#### 3. Write Component Files

##### Detect Existing Directory Structure

Before writing files, detect existing React project conventions:

```bash
# React/Next.js: Check for existing component directories
Glob("src/components/**/*.tsx") || Glob("components/**/*.tsx") || Glob("app/components/**/*.tsx")
```

Use the detected structure to determine where to place new components. If no existing structure is found, use the default structure below.

##### React/Next.js Directory Structure

```
src/
├── components/
│   ├── ui/                    # Atomic components
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── Badge.tsx
│   └── {feature}/             # Feature components
│       ├── HeroSection.tsx
│       └── NavigationBar.tsx
└── styles/
    └── tokens.css             # CSS custom properties
```

## React Component Structure

### Component Example

```tsx
import React from 'react';
import { cn } from '@/lib/utils';

export interface CardProps {
  /** Card title */
  title: string;
  /** Card description */
  description?: string;
  /** Image source URL */
  imageSrc?: string;
  /** Image alt text */
  imageAlt?: string;
  /** Additional CSS classes */
  className?: string;
  /** Child elements */
  children?: React.ReactNode;
}

export const Card: React.FC<CardProps> = ({
  title,
  description,
  imageSrc,
  imageAlt = '',
  className,
  children,
}) => {
  return (
    <article
      className={cn(
        'flex flex-col rounded-lg bg-[var(--color-card)] shadow-md',
        'p-6 gap-4',
        className
      )}
    >
      {imageSrc && (
        <img
          src={imageSrc}
          alt={imageAlt}
          className="w-full h-48 object-cover rounded-md"
        />
      )}
      <header>
        <h3 className="text-xl font-semibold text-[var(--color-text)]">
          {title}
        </h3>
        {description && (
          <p className="mt-2 text-[var(--color-text-muted)]">
            {description}
          </p>
        )}
      </header>
      {children && <div className="mt-auto">{children}</div>}
    </article>
  );
};

export default Card;
```

### Component Variants with CVA

For components with multiple variants (size, color, state), use class-variance-authority (cva):

**Install:**
```bash
npm install class-variance-authority
```

**Pattern:**
```tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

const buttonVariants = cva(
  // Base classes (always applied)
  "inline-flex items-center justify-center rounded-md font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none",
  {
    variants: {
      variant: {
        primary: "bg-primary text-white hover:bg-primary/90 focus:ring-primary",
        secondary: "bg-secondary text-white hover:bg-secondary/90 focus:ring-secondary",
        outline: "border border-primary text-primary hover:bg-primary/10",
        ghost: "text-primary hover:bg-primary/10",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4 text-base",
        lg: "h-12 px-6 text-lg",
      },
    },
    defaultVariants: {
      variant: "primary",
      size: "md",
    },
  }
);

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  children: React.ReactNode;
}

export const Button: React.FC<ButtonProps> = ({
  variant,
  size,
  className,
  children,
  ...props
}) => {
  return (
    <button
      className={cn(buttonVariants({ variant, size }), className)}
      {...props}
    >
      {children}
    </button>
  );
};

// Usage
<Button variant="primary" size="lg">Click me</Button>
<Button variant="outline">Outlined</Button>
```

**When to use CVA:**

| Scenario | Use CVA? | Reason |
|----------|----------|--------|
| 2+ variants defined in spec | Yes | Cleaner than conditionals |
| Single variant | No | Simple conditional or prop |
| Complex state combinations | Yes | Manages compound variants |
| One-off component | No | Overkill for simple cases |

**Extract variants from spec:**

```markdown
## Components

### Button
| Property | Value |
|----------|-------|
| **Variants** | primary, secondary, outline, ghost |
| **Sizes** | sm (32px), md (40px), lg (48px) |
```

→ Map directly to CVA variants object.

### Component Checklist

For each generated component, verify:

**Structure & Hierarchy:**
- [ ] **Hierarchy matches spec** - Component structure follows the spec hierarchy
- [ ] **Semantic HTML used** - Proper elements (button, nav, article, section, etc.)
- [ ] **TypeScript types** - Interface defined with proper prop types and JSDoc

**Design Tokens:**
- [ ] **Colors applied** - Uses CSS variables or Tailwind tokens from spec
- [ ] **Opacity applied** - Includes `/opacity` modifiers where spec shows < 1.0
- [ ] **Typography correct** - Font size, weight, line-height match spec

**Frame Properties:**
- [ ] **Dimensions applied** - `w-[Xpx]` or `max-w-[Xpx]` from spec Dimensions
- [ ] **Corner radius applied** - `rounded-*` classes match spec Corner Radius
- [ ] **Border applied** - `border` classes with correct color/opacity

**Visual Effects:**
- [ ] **Gradients rendered** - `bg-[linear-gradient(...)]` or style prop for complex
- [ ] **Text decoration** - `underline`/`line-through` with correct color
- [ ] **Shadows applied** - `shadow-*` classes match spec effects

**Assets:**
- [ ] **Assets referenced** - Images/icons use paths from Downloaded Assets
- [ ] **Asset Node Map followed** - IMAGE: entries converted to proper img/Image
- [ ] **Alt text provided** - All images have descriptive alt attributes
- [ ] **Icon pattern correct** - lucide-react, SVG component, or img based on needs

**Accessibility:**
- [ ] **ARIA attributes** - aria-label, aria-describedby where needed
- [ ] **Focus states** - `focus:ring-*` or `focus-visible:*` classes
- [ ] **Keyboard navigable** - Interactive elements accessible via keyboard

**Code Quality:**
- [ ] **cn() used** - Class names merged with cn() utility
- [ ] **Props typed** - All props have TypeScript types
- [ ] **Default exports** - Named export + default export pattern

## React-Specific Error Handling

### Type Errors

1. Identify the type error from linter/compiler output
2. Fix the type definition
3. Re-validate with TypeScript compiler:
   ```bash
   npx tsc --noEmit {file_path}
   ```
4. If errors persist:
   - Document in Generated Code table with status "WARN"
   - Add fix instructions in summary

### Missing Assets

1. Check if asset exists in Downloaded Assets section
2. If missing:
   - Use placeholder path: `/assets/placeholder.svg`
   - Add TODO comment in code:
     ```tsx
     {/* TODO: Replace with actual asset path */}
     <img src="/assets/placeholder.svg" alt="[Asset name]" />
     ```
   - Document in Generated Code table with status "WARN - Missing asset"
   - Add to summary: "Asset {name} not found - using placeholder"

## Manual Generation Fallback

When MCP generation is unavailable, generate React code from spec:

### Extract from Spec

1. **Component properties** from Components section
2. **Layout classes** from Classes/Styles field
3. **Semantic element** from Element field
4. **Children** from Children field
5. **Design tokens** from Design Tokens (Ready to Use) section

### Generate React Structure

```tsx
// From spec:
// Element: <section>
// Layout: flex flex-col
// Classes: p-6 gap-4 bg-white rounded-lg shadow-md

export const {ComponentName}: React.FC<{ComponentName}Props> = ({
  // props from spec
}) => {
  return (
    <section className="flex flex-col p-6 gap-4 bg-white rounded-lg shadow-md">
      {/* Children from spec */}
    </section>
  );
};
```

## React Guidelines

### Naming Conventions

- **Component names**: PascalCase (e.g., `HeroSection`, `NavigationBar`)
- **File names**: Match component name (e.g., `HeroSection.tsx`)
- **Props interfaces**: ComponentNameProps (e.g., `HeroSectionProps`)

### Code Quality Standards

- Use TypeScript strict mode
- Include JSDoc comments for props
- Extract repeated styles to utility classes
- Keep components focused and single-purpose
- Use composition over complex conditional rendering

### Tailwind Best Practices

- Use design tokens over arbitrary values when possible
- Group related utilities (layout, spacing, colors, effects)
- Use `cn()` utility for conditional classes
- Prefer responsive prefixes over media queries

### Accessibility Requirements

- All images must have alt text
- Interactive elements must be focusable
- Color contrast must meet WCAG AA (4.5:1)
- Include focus-visible styles for keyboard users
- Use semantic HTML elements appropriately

## Responsive Patterns

Map Figma constraints and breakpoints to Tailwind responsive prefixes.

### Figma Constraints → Tailwind

| Figma Constraint | Tailwind Pattern |
|------------------|------------------|
| Fill container (width) | `w-full` |
| Fixed width | `w-[Xpx]` |
| Hug contents | `w-auto` or `w-fit` |
| Min/Max width | `min-w-[X] max-w-[Y]` |

### Breakpoint Mapping

**Standard Figma frame sizes:**

| Figma Frame | Tailwind Breakpoint | Use |
|-------------|---------------------|-----|
| 375px (Mobile) | Default (no prefix) | Mobile-first base |
| 768px (Tablet) | `md:` | Tablet layouts |
| 1024px (Desktop) | `lg:` | Desktop layouts |
| 1280px (Large) | `xl:` | Wide screens |

**Responsive component example:**

```tsx
// Card that adapts to screen size
<article className={cn(
  // Mobile (default): Full width, vertical
  "flex flex-col w-full gap-4 p-4",
  // Tablet: Horizontal layout, max width
  "md:flex-row md:max-w-[600px] md:p-6",
  // Desktop: Larger padding
  "lg:max-w-[800px] lg:p-8"
)}>
  <Image
    src={imageSrc}
    alt={imageAlt}
    // Responsive image sizing
    className="w-full h-48 md:w-1/3 md:h-auto object-cover rounded-lg"
  />
  <div className="flex-1">
    <h2 className="text-lg md:text-xl lg:text-2xl font-bold">{title}</h2>
    <p className="text-sm md:text-base text-gray-600">{description}</p>
  </div>
</article>
```

### Auto Layout → Flexbox/Grid

| Figma Auto Layout | Tailwind |
|-------------------|----------|
| Horizontal | `flex flex-row` |
| Vertical | `flex flex-col` |
| Wrap | `flex flex-wrap` |
| Gap: 16 | `gap-4` |
| Padding: 24 | `p-6` |
| Space between | `justify-between` |
| Align center | `items-center` |

**Grid for complex layouts:**
```tsx
// 2-column grid on desktop, single column on mobile
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {items.map(item => <Card key={item.id} {...item} />)}
</div>
```

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

## Required Utilities

**CRITICAL:** When generating React code, include these helper utilities if needed.

### cn() Utility (Class Name Merger)

If any generated code uses `cn()`, ensure this utility exists:

```tsx
// lib/utils.ts
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**Dependencies required:**
```bash
npm install clsx tailwind-merge
```

**Usage:**
```tsx
import { cn } from '@/lib/utils';

<div className={cn(
  "flex flex-col",           // Base classes
  isActive && "bg-primary",  // Conditional
  className                  // Props override
)}>
```

### CSS Variables Setup

If spec uses CSS custom properties, ensure they're defined:

```css
/* styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --color-primary: #3B82F6;
    --color-secondary: #6B7280;
    --color-background: #FFFFFF;
    --color-foreground: #1F2937;
    --color-border: #E5E7EB;
    /* From Design Tokens section */
  }

  .dark {
    --color-primary: #60A5FA;
    --color-secondary: #9CA3AF;
    --color-background: #111827;
    --color-foreground: #F9FAFB;
    --color-border: #374151;
  }
}
```

**Usage in Tailwind:**
```tsx
<div className="bg-[var(--color-background)] text-[var(--color-foreground)]">
```

### Tailwind 4 Theme Setup (if using Tailwind v4)

```css
/* styles/globals.css */
@import "tailwindcss";

@theme {
  --color-primary: #3B82F6;
  --color-secondary: #6B7280;
  --color-accent: #F59E0B;
  /* Semantic colors from Design Tokens */

  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;

  --radius-sm: 0.25rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
  --radius-xl: 0.75rem;
}
```

### When to Include Utilities

| Utility | Include When |
|---------|--------------|
| cn() | Any conditional class merging |
| CSS Variables | Design tokens used with `var(--...)` |
| Tailwind @theme | Tailwind v4 project with custom tokens |

**Check for existing setup:**
```bash
# Check if cn exists
Grep("export function cn", path="lib/utils.ts")
Grep("export function cn", path="src/lib/utils.ts")

# Check if CSS variables exist
Grep("--color-primary", path="styles/globals.css")
```

## Output

Update the Implementation Spec at: `docs/figma-reports/{file_key}-spec.md`

### Sections Added to Spec

```markdown
## Generated Code

| Component | File | Status |
|-----------|------|--------|
| Card | `src/components/ui/Card.tsx` | OK |
| Button | `src/components/ui/Button.tsx` | OK |
| HeroSection | `src/components/HeroSection.tsx` | OK |
| NavigationBar | `src/components/NavigationBar.tsx` | WARN - Manual adjustments needed |

## Code Generation Summary

- **Framework:** React/Next.js + Tailwind
- **Components generated:** {count}
- **Files created:** {count}
- **Warnings:** {count}
- **Generation timestamp:** {YYYYMMDD-HHmmss}

## Files Created

### Components
- `src/components/ui/Card.tsx`
- `src/components/ui/Button.tsx`
- `src/components/HeroSection.tsx`

### Styles (if created)
- `src/styles/tokens.css`

## Next Agent Input

Ready for: Compliance Checker Agent
Input file: `docs/figma-reports/{file_key}-spec.md`
Components generated: {count}
Framework: React/Next.js + Tailwind
```
