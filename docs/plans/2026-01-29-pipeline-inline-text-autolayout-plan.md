# Pipeline Inline Text, Text Sizing & Adaptive Layout — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add characterStyleOverrides detection, textAutoResize support, and tablet-adaptive layout patterns to the pb-figma 5-agent pipeline.

**Architecture:** Additive changes to 4 agent markdown files. No existing behavior removed. Changes flow through the pipeline: design-validator detects → design-analyst documents → code-generator uses → compliance-checker verifies.

**Tech Stack:** Figma REST API properties (characterStyleOverrides, styleOverrideTable, textAutoResize, layoutMode, constraints), SwiftUI adaptive patterns (horizontalSizeClass, maxWidth, LazyVGrid)

**Design Doc:** `docs/plans/2026-01-29-pipeline-inline-text-autolayout-design.md`

---

## Task 1: Add characterStyleOverrides detection to design-validator

**Files:**
- Modify: `plugins/pb-figma/agents/design-validator.md:49-56` (Design Tokens checklist)
- Modify: `plugins/pb-figma/agents/design-validator.md:284-289` (Typography section in report template)

**Step 1: Add checklist item for inline text variations**

In `design-validator.md`, after line 56 (`- [ ] Effects documented (shadows, blurs)`), add a new checklist item:

```markdown
- [ ] **Inline text variations detected** (characterStyleOverrides on TEXT nodes)
```

**Step 2: Add detection logic section**

After the existing `### 3.6 Fill Opacity Extraction` section (which ends around line 283), add a new section `### 3.7 Inline Text Variation Detection`:

```markdown
### 3.7 Inline Text Variation Detection

**Problem:** A single TEXT node may have multiple character styles (different colors, weights, or decorations for different words). The REST API exposes this via `characterStyleOverrides` and `styleOverrideTable`.

**Detection Pattern:**

```typescript
const nodeDetails = figma_get_node_details({
  file_key: "{file_key}",
  node_id: "{text_node_id}"
});

// Check for character-level style overrides
const overrides = nodeDetails.characterStyleOverrides;
if (overrides && overrides.length > 0 && overrides.some(v => v !== 0)) {
  // Text has inline style variations
  // Count unique non-zero override indices
  const uniqueStyles = [...new Set(overrides.filter(v => v !== 0))];

  // Extract style details from styleOverrideTable
  const styleTable = nodeDetails.styleOverrideTable || {};
  const styleDetails = uniqueStyles.map(idx => {
    const style = styleTable[idx];
    if (!style || Object.keys(style).length === 0) {
      return `${idx}: (default style — known API bug)`;
    }
    const fills = style.fills?.map(f => {
      const hex = rgbToHex(f.color.r, f.color.g, f.color.b);
      return hex;
    }).join(', ') || 'inherited';
    const decoration = style.textDecoration || 'NONE';
    return `${idx}: fills: ${fills}, decoration: ${decoration}`;
  });

  // Flag for design-analyst
  return {
    nodeId: text_node_id,
    text: nodeDetails.characters,
    overrideCount: overrides.filter(v => v !== 0).length,
    uniqueStyles: styleDetails
  };
}
```

**When to check:** For EVERY TEXT node encountered during validation, query `characterStyleOverrides`. This is lightweight (data is already in the node details response).

**Known Figma API Bug:** `styleOverrideTable` may return empty objects `{}` for default-value overrides. Document these as `(default style)` — the design-analyst will use the node's base style for those characters.
```

**Step 3: Add report output section**

In the Validation Report template (after the Typography table around line 289), add:

```markdown
### Inline Text Variations Detected

| Node ID | Text Content | Override Count | Unique Styles |
|---------|-------------|----------------|---------------|
| {id}    | "{text}"    | {count} chars  | {style_details} |

> **Note:** Empty style objects `{}` in styleOverrideTable indicate default-value overrides (known Figma API behavior). These characters use the node's base text style.
```

**Step 4: Commit**

```bash
git add plugins/pb-figma/agents/design-validator.md
git commit -m "feat(pb-figma): add characterStyleOverrides detection to design-validator

Detects inline text variations (different colors/weights/decorations
within a single TEXT node) via Figma REST API characterStyleOverrides.
Documents findings in validation report for design-analyst consumption."
```

---

## Task 2: Add Figma API empty override bug note to design-analyst

**Files:**
- Modify: `plugins/pb-figma/agents/design-analyst.md:579-591` (characterStyleOverrides code block)
- Modify: `plugins/pb-figma/agents/design-analyst.md:634-639` (Detection Rules)

**Step 1: Add empty object handling to the code example**

In `design-analyst.md`, replace the existing code block at lines 579-591:

```typescript
// Check if text has character-level style overrides
if (nodeDetails.characterStyleOverrides?.length > 0) {
  // Text has inline style variations
  // Cross-reference with styleOverrideTable for actual styles
  const styleTable = nodeDetails.styleOverrideTable;

  for (const [key, style] of Object.entries(styleTable)) {
    // Each style may have different fills (colors)
    const fills = style.fills;
    // Extract color for each style variation
  }
}
```

With this updated version that handles the empty object bug:

```typescript
// Check if text has character-level style overrides
if (nodeDetails.characterStyleOverrides?.length > 0) {
  // Text has inline style variations
  // Cross-reference with styleOverrideTable for actual styles
  const styleTable = nodeDetails.styleOverrideTable;

  for (const [key, style] of Object.entries(styleTable)) {
    // KNOWN BUG: styleOverrideTable may return {} for default-value overrides
    // When style is empty object, use node's base style instead
    if (Object.keys(style).length === 0) {
      // Empty override = default style (use node's fills, fontSize, fontWeight)
      continue; // Characters with this override use base text style
    }

    // Each style may have different fills (colors)
    const fills = style.fills;
    // Extract color for each style variation
  }
}
```

**Step 2: Add empty override handling to Detection Rules**

In `design-analyst.md`, after line 638 (`4. If API doesn't support, flag for visual inspection`), add a new rule:

```markdown
5. If `styleOverrideTable` contains empty objects `{}` for an override index, treat those characters as using the node's base style (fills, fontSize, fontWeight from the TEXT node itself). This is a known Figma REST API behavior where default-value overrides return empty objects instead of explicit values.
```

**Step 3: Commit**

```bash
git add plugins/pb-figma/agents/design-analyst.md
git commit -m "fix(pb-figma): handle empty styleOverrideTable entries in design-analyst

Adds handling for known Figma API bug where styleOverrideTable returns
empty objects {} for default-value overrides. Characters with empty
overrides now use the TEXT node's base style."
```

---

## Task 3: Add textAutoResize and frame dimensions to design-analyst

**Files:**
- Modify: `plugins/pb-figma/agents/design-analyst.md:516-563` (Text Decoration Detection section area)

**Step 1: Add Text Sizing Detection section**

In `design-analyst.md`, after the `#### Inline Text Style Variations Detection` section (which ends at line 639), add a new section before `### 4. Asset Requirements` (line 640):

```markdown
#### Text Sizing & Auto-Resize Detection

**Problem:** Without knowing how Figma sizes a text frame, the code generator cannot determine if text should wrap, truncate, or auto-size.

**Detection via Figma API:**

When `figma_get_node_details` returns a TEXT node, extract these sizing properties:

```typescript
const nodeDetails = figma_get_node_details({
  file_key: "{file_key}",
  node_id: "{text_node_id}"
});

// Text auto-resize mode
const textAutoResize = nodeDetails.textAutoResize;
// Values: "NONE" | "HEIGHT" | "WIDTH_AND_HEIGHT" | "TRUNCATE"

// Text frame dimensions
const frameWidth = nodeDetails.absoluteBoundingBox?.width;
const frameHeight = nodeDetails.absoluteBoundingBox?.height;

// Font properties for line count calculation
const fontSize = nodeDetails.style?.fontSize;
const lineHeightPx = nodeDetails.style?.lineHeightPx;
const lineHeight = lineHeightPx || (fontSize * 1.2); // fallback

// Approximate line count
const lineCountHint = Math.floor(frameHeight / lineHeight);
```

**Implementation Spec Output:**

For each TEXT node, add sizing properties to the component spec:

```markdown
| Property | Value |
|----------|-------|
| **Auto-Resize** | `HEIGHT` |
| **Frame Dimensions** | `311 × 38` |
| **Line Count Hint** | `2` (based on frameHeight / lineHeight) |
```

**Auto-Resize Values and SwiftUI Mapping:**

| Figma Value | Behavior | SwiftUI |
|-------------|----------|---------|
| `NONE` | Fixed frame, text may clip | `.frame(width:height:).clipped()` |
| `HEIGHT` | Width fixed, height adjusts | No `.lineLimit()` needed (default) |
| `WIDTH_AND_HEIGHT` | Both adjust | No constraints needed |
| `TRUNCATE` | Fixed frame, text truncated | `.lineLimit(N)` where N = lineCountHint |

**Rules:**
1. Always extract `textAutoResize` for TEXT nodes
2. Always include `absoluteBoundingBox` width and height for text frames
3. Calculate `lineCountHint` when `textAutoResize` is `HEIGHT` or `TRUNCATE`
4. If `lineHeightPx` is not available, use `fontSize * 1.2` as fallback
5. Add these properties to the component property table in the spec
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/design-analyst.md
git commit -m "feat(pb-figma): add textAutoResize and frame dimensions to design-analyst

Extracts text sizing mode (NONE/HEIGHT/WIDTH_AND_HEIGHT/TRUNCATE),
frame dimensions, and line count hint for TEXT nodes. Enables
code-generator to properly handle text wrapping and truncation."
```

---

## Task 4: Add text auto-resize handling to code-generator-swiftui

**Files:**
- Modify: `plugins/pb-figma/agents/code-generator-swiftui.md:1173-1179` (after the inline text variations section)

**Step 1: Add Text Sizing section**

In `code-generator-swiftui.md`, after the "Common mistakes" block at line 1179, add a new section:

```markdown
##### Apply Text Sizing from Spec

**Read from Implementation Spec:**

Check for text sizing properties in each component's property table:

```markdown
| Property | Value |
|----------|-------|
| **Auto-Resize** | `HEIGHT` |
| **Frame Dimensions** | `311 × 38` |
| **Line Count Hint** | `2` |
```

**SwiftUI Code Generation by Auto-Resize Mode:**

**1. `HEIGHT` (width fixed, height adjusts):**
```swift
// Default behavior — no lineLimit needed
Text("Description text that may wrap to multiple lines")
    .font(.system(size: 14))
    .foregroundColor(.white.opacity(0.7))
// Do NOT add .lineLimit() — let text wrap naturally
```

**2. `TRUNCATE` (fixed frame, text truncated):**
```swift
Text("Text that should truncate if too long")
    .font(.system(size: 14))
    .foregroundColor(.white.opacity(0.7))
    .lineLimit(2) // lineCountHint from spec
    .truncationMode(.tail)
```

**3. `NONE` (fixed frame, text may clip):**
```swift
Text("Fixed frame text")
    .font(.system(size: 14))
    .foregroundColor(.white.opacity(0.7))
    .frame(width: 311, height: 38) // exact frame dimensions from spec
    .clipped()
```

**4. `WIDTH_AND_HEIGHT` (both dimensions adjust):**
```swift
// No constraints — text sizes to content
Text("Auto-sizing text")
    .font(.system(size: 14))
    .foregroundColor(.white.opacity(0.7))
    .fixedSize(horizontal: false, vertical: true) // ensure no implicit truncation
```

**Rules:**
1. If `Auto-Resize` is `HEIGHT` → do NOT add `.lineLimit()` (let text wrap freely)
2. If `Auto-Resize` is `TRUNCATE` → add `.lineLimit(N)` using `Line Count Hint` value
3. If `Auto-Resize` is `NONE` → add `.frame(width:height:)` and `.clipped()`
4. If `Auto-Resize` is `WIDTH_AND_HEIGHT` → add `.fixedSize(horizontal: false, vertical: true)`
5. If no `Auto-Resize` property in spec → default to `HEIGHT` behavior (no lineLimit)
6. When `Line Count Hint` > 1 and no explicit `Auto-Resize` → ensure no `.lineLimit(1)` is applied

**Common mistakes:**

❌ Adding `.lineLimit(1)` when Auto-Resize is `HEIGHT` → Text truncated unexpectedly
✅ No `.lineLimit()` when Auto-Resize is `HEIGHT` → Text wraps naturally

❌ Missing `.truncationMode(.tail)` with `.lineLimit()` → Default may vary
✅ Always pair `.lineLimit(N)` with `.truncationMode(.tail)` for TRUNCATE mode

❌ Using `.frame(maxHeight:)` for `NONE` mode → Text may overflow
✅ Using `.frame(width:height:).clipped()` for `NONE` mode → Clean clipping
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-swiftui.md
git commit -m "feat(pb-figma): add text auto-resize handling to code-generator-swiftui

Maps Figma textAutoResize modes (NONE/HEIGHT/TRUNCATE/WIDTH_AND_HEIGHT)
to SwiftUI modifiers (lineLimit, frame+clipped, fixedSize). Prevents
unintended text truncation."
```

---

## Task 5: Add Auto Layout property detection to design-validator

**Files:**
- Modify: `plugins/pb-figma/agents/design-validator.md:43-47` (Structure checklist)
- Modify: `plugins/pb-figma/agents/design-validator.md:193-283` (after Frame Properties section)

**Step 1: Expand Structure checklist**

In `design-validator.md`, replace line 47:

```markdown
- [ ] Auto Layout is used (WARN if absolute positioning)
```

With expanded checklist items:

```markdown
- [ ] Auto Layout is used (WARN if absolute positioning)
- [ ] **Auto Layout properties extracted** (layoutMode, axis alignment, padding, spacing, constraints)
- [ ] **Min/Max dimensions captured** (minWidth, maxWidth, minHeight, maxHeight)
```

**Step 2: Add Auto Layout extraction section**

After the `### 3.6 Fill Opacity Extraction` section (and after the new `### 3.7 Inline Text Variation Detection` section from Task 1), add:

```markdown
### 3.8 Auto Layout Property Extraction

**CRITICAL:** Extract Auto Layout properties for ALL frames that use Auto Layout (`layoutMode` ≠ "NONE").

**Query Pattern:**

```typescript
const nodeDetails = figma_get_node_details({
  file_key: "{file_key}",
  node_id: "{frame_node_id}"
});

// Auto Layout detection
const layoutMode = nodeDetails.layoutMode; // "NONE" | "HORIZONTAL" | "VERTICAL"

if (layoutMode !== "NONE") {
  // Axis alignment
  const primaryAxisAlign = nodeDetails.primaryAxisAlignItems;   // MIN | CENTER | MAX | SPACE_BETWEEN
  const counterAxisAlign = nodeDetails.counterAxisAlignItems;   // MIN | CENTER | MAX | BASELINE

  // Padding (individual sides)
  const paddingTop = nodeDetails.paddingTop ?? 0;
  const paddingRight = nodeDetails.paddingRight ?? 0;
  const paddingBottom = nodeDetails.paddingBottom ?? 0;
  const paddingLeft = nodeDetails.paddingLeft ?? 0;

  // Item spacing
  const itemSpacing = nodeDetails.itemSpacing ?? 0;

  // Constraints (responsive behavior)
  const constraints = nodeDetails.constraints; // { horizontal, vertical }
  // Values: "MIN" | "CENTER" | "MAX" | "STRETCH" | "SCALE"

  // Min/Max dimensions
  const minWidth = nodeDetails.minWidth;
  const maxWidth = nodeDetails.maxWidth;
  const minHeight = nodeDetails.minHeight;
  const maxHeight = nodeDetails.maxHeight;
}
```

**Auto Layout Properties Table in Validation Report:**

```markdown
### Auto Layout Properties

| Node ID | Node Name | Layout Mode | Primary Axis | Counter Axis | Padding (T/R/B/L) | Spacing | Constraints (H/V) | Min/Max Width |
|---------|-----------|-------------|-------------|--------------|-------------------|---------|-------------------|---------------|
| 3:100 | ContentArea | VERTICAL | MIN | CENTER | 16/16/16/16 | 16 | STRETCH/MIN | -/- |
| 3:200 | CardRow | HORIZONTAL | MIN | CENTER | 16/16/16/16 | 16 | STRETCH/MIN | -/- |
```

**Padding Format:** `T/R/B/L` (e.g., `16/16/16/16` for uniform, `24/16/24/16` for different sides)

**Constraints Format:** `H/V` (e.g., `STRETCH/MIN`)

**Min/Max Format:** `{min}/{max}` or `-` if not set

**Rules:**
1. Only include frames where `layoutMode` ≠ "NONE"
2. Extract ALL Auto Layout properties — do not omit padding or constraints
3. Document uniform padding as `16/16/16/16` (not just `16`)
4. Include constraints to inform responsive behavior
5. Min/Max dimensions inform tablet adaptive patterns
```

**Step 3: Commit**

```bash
git add plugins/pb-figma/agents/design-validator.md
git commit -m "feat(pb-figma): add Auto Layout property extraction to design-validator

Extracts layoutMode, axis alignment, padding, spacing, constraints,
and min/max dimensions for Auto Layout frames. Provides data for
adaptive layout generation."
```

---

## Task 6: Add Auto Layout info to design-analyst spec output

**Files:**
- Modify: `plugins/pb-figma/agents/design-analyst.md:262-276` (Example Component section)
- Modify: `plugins/pb-figma/agents/design-analyst.md:293-307` (CSS/Responsive section)

**Step 1: Expand component property table example**

In `design-analyst.md`, replace the example component at lines 262-276:

```markdown
**Example Component with Complete Dimensions:**

```markdown
### RoadmapCard

| Property | Value |
|----------|-------|
| **Element** | HStack |
| **Layout** | horizontal, spacing: 16 |
| **Dimensions** | `width: 358, height: 64` |  ← MUST include height
| **Corner Radius** | `32px` |
| **Border** | `1px #414141 inside` |
| **Children** | IconFrame, TitleText, CheckmarkIcon |
| **Asset Children** | `IMAGE:icon-clock:3:230:32:32` |
```
```

With expanded version including Auto Layout:

```markdown
**Example Component with Complete Dimensions and Auto Layout:**

```markdown
### RoadmapCard

| Property | Value |
|----------|-------|
| **Element** | HStack |
| **Layout** | horizontal, spacing: 16 |
| **Dimensions** | `width: 358, height: 64` |  ← MUST include height
| **Auto Layout** | `HORIZONTAL, primaryAxis: MIN, counterAxis: CENTER` |
| **Padding** | `16/16/16/16` (T/R/B/L) |
| **Constraints** | `horizontal: STRETCH, vertical: MIN` |
| **Corner Radius** | `32px` |
| **Border** | `1px #414141 inside` |
| **Children** | IconFrame, TitleText, CheckmarkIcon |
| **Asset Children** | `IMAGE:icon-clock:3:230:32:32` |
```
```

**Step 2: Add Auto Layout reading instructions**

After section `### 1.5 Read Frame Properties from Validation Report` (around line 220), add a new section:

```markdown
### 1.6 Read Auto Layout Properties from Validation Report

**Process:**
1. Find component's Node ID in Validation Report "### Auto Layout Properties" table
2. Extract: Layout Mode, Primary/Counter Axis, Padding, Spacing, Constraints, Min/Max
3. Add to component's property table in spec

**Mapping to SwiftUI:**

| Auto Layout Property | SwiftUI Equivalent |
|---------------------|--------------------|
| HORIZONTAL | `HStack(spacing: {itemSpacing})` |
| VERTICAL | `VStack(spacing: {itemSpacing})` |
| primaryAxis: MIN | `.frame(maxWidth: .infinity, alignment: .leading)` |
| primaryAxis: CENTER | `.frame(maxWidth: .infinity, alignment: .center)` |
| primaryAxis: SPACE_BETWEEN | `Spacer()` between children |
| counterAxis: CENTER | Default VStack/HStack alignment |
| Padding T/R/B/L | `.padding(EdgeInsets(top:leading:bottom:trailing:))` |
| constraints: STRETCH | `.frame(maxWidth: .infinity)` |
| constraints: MIN | No width constraint (hug contents) |

**Responsive Hints:**

When a frame has `constraints.horizontal: STRETCH` AND no `maxWidth`:
- Add to spec: `Responsive: content stretches to fill parent`
- Code generator should use `.frame(maxWidth: .infinity)`

When a frame has `maxWidth` set:
- Add to spec: `Responsive: max width {maxWidth}pt`
- Code generator should use `.frame(maxWidth: {maxWidth})`
```

**Step 3: Commit**

```bash
git add plugins/pb-figma/agents/design-analyst.md
git commit -m "feat(pb-figma): add Auto Layout properties to design-analyst spec

Reads Auto Layout properties from validation report and maps them to
SwiftUI equivalents. Includes padding, constraints, and responsive hints
for adaptive layout generation."
```

---

## Task 7: Add adaptive layout patterns to code-generator-swiftui

**Files:**
- Modify: `plugins/pb-figma/agents/code-generator-swiftui.md` (add new section after text handling)

**Step 1: Add Adaptive Layout section**

In `code-generator-swiftui.md`, find the end of the text handling sections (after the new Text Sizing section from Task 4). Add a new top-level section:

```markdown
#### Adaptive Layout Patterns (iPad/Tablet Support)

**Read from Implementation Spec:**

Check for Auto Layout and responsive properties in component specs:

```markdown
| Property | Value |
|----------|-------|
| **Auto Layout** | `VERTICAL, primaryAxis: MIN, counterAxis: CENTER` |
| **Constraints** | `horizontal: STRETCH, vertical: MIN` |
| **Responsive** | `content stretches to fill parent` |
```

**Rule 1 — Content Width Cap:**

All top-level content containers (the outermost VStack/ScrollView) MUST include a width cap for iPad readability:

```swift
VStack(spacing: 16) {
    // content
}
.frame(maxWidth: 600) // prevent over-stretching on iPad
.frame(maxWidth: .infinity) // center within parent
```

**When to apply:**
- The root container of ANY generated view
- Only at the top level (not nested containers)

**Rule 2 — Card Lists with 3+ Items:**

When a VStack contains 3 or more card-like children with identical structure (same component type repeated), use adaptive grid:

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

**When to apply:**
- Spec shows a repeating card pattern (ForEach over items)
- 3+ items with identical structure
- NOT for 1-2 items (keep VStack)

**Rule 3 — Safe Layout Defaults:**

ALWAYS follow these defaults in generated code:

```swift
// ✅ DO: Use flexible widths
.frame(maxWidth: .infinity)

// ❌ DON'T: Use screen-dependent widths
.frame(width: UIScreen.main.bounds.width)
.frame(width: 393) // hardcoded iPhone width

// ✅ DO: Use horizontal padding for edge spacing
.padding(.horizontal, 16)

// ❌ DON'T: Calculate padding from screen width
.padding(.horizontal, (UIScreen.main.bounds.width - 361) / 2)
```

**Rule 4 — Size Class (Optional, for complex layouts):**

Only use when the spec explicitly mentions different tablet layout OR the design has major structural differences for wider screens:

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

**Common mistakes:**

❌ Hardcoding `UIScreen.main.bounds.width` → Breaks on iPad and landscape
✅ Using `.frame(maxWidth: .infinity)` → Works on all screen sizes

❌ Missing maxWidth cap on root container → Content stretches too wide on iPad
✅ `.frame(maxWidth: 600).frame(maxWidth: .infinity)` → Capped and centered

❌ Using VStack for 5+ repeating cards → Wasted horizontal space on iPad
✅ Using `LazyVGrid(.adaptive(...))` → Auto-adjusts columns by screen width
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-swiftui.md
git commit -m "feat(pb-figma): add adaptive layout patterns to code-generator-swiftui

Adds content width cap, adaptive grid for card lists, safe layout
defaults, and optional size class detection for tablet support.
Prevents hardcoded screen widths and ensures iPad compatibility."
```

---

## Task 8: Add tablet layout verification to compliance-checker

**Files:**
- Modify: `plugins/pb-figma/agents/compliance-checker.md:257-264` (after Responsive Verification section header)

**Step 1: Add Tablet Layout Verification section**

In `compliance-checker.md`, find the end of the existing compliance checks (after section `### 7. Responsive Verification`). Add a new section:

```markdown
### 8. Tablet Layout Verification (SwiftUI)

Verify generated SwiftUI code follows adaptive layout patterns:

- [ ] **Content width capped** - Root container has `.frame(maxWidth: N)` where N ≤ 700
- [ ] **No hardcoded screen widths** - No `UIScreen.main.bounds.width` references
- [ ] **No hardcoded iPhone widths** - No `.frame(width: 393)` or similar fixed widths
- [ ] **Card lists adaptive** - Lists with 3+ repeating items use `LazyVGrid(.adaptive(...))` or justify VStack usage
- [ ] **Flexible widths used** - Containers use `.frame(maxWidth: .infinity)` not fixed widths
- [ ] **Padding uses system values** - `.padding(.horizontal, N)` not calculated from screen width

**Verification Method (SwiftUI):**
```
# Check for hardcoded screen widths (should NOT exist)
Grep("UIScreen\\.main\\.bounds", path="{component_file_path}")

# Check for content width cap (SHOULD exist on root view)
Grep("\\.frame\\(maxWidth:", path="{component_file_path}")

# Check for fixed iPhone widths (should NOT exist)
Grep("\\.frame\\(width: 393\\)", path="{component_file_path}")
Grep("\\.frame\\(width: 390\\)", path="{component_file_path}")

# Check for adaptive grid usage (should exist for card lists)
Grep("LazyVGrid\\|GridItem\\|adaptive", path="{component_file_path}")
```

**Severity Levels:**
- `UIScreen.main.bounds` usage → **HIGH** (breaks on iPad)
- Missing maxWidth cap on root → **MEDIUM** (content stretches)
- Fixed width matching iPhone → **MEDIUM** (layout breaks on larger screens)
- VStack for 3+ repeating items → **LOW** (functional but suboptimal)

**Report Output:**

```markdown
### Tablet Layout Verification

| Check | Status | Details |
|-------|--------|---------|
| Content width cap | ✅/❌ | maxWidth: {value} on root container |
| No hardcoded screen widths | ✅/❌ | Found {count} UIScreen references |
| Adaptive grid for cards | ✅/❌ | {count} card lists using LazyVGrid |
| Flexible widths | ✅/❌ | {count} fixed widths found |
```
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/compliance-checker.md
git commit -m "feat(pb-figma): add tablet layout verification to compliance-checker

Checks for content width caps, hardcoded screen widths, adaptive grid
usage, and flexible width patterns. Ensures generated SwiftUI code
works on iPad and larger screens."
```

---

## Summary

| Task | Agent | Change | Commit Message |
|------|-------|--------|----------------|
| 1 | design-validator | characterStyleOverrides detection | `feat: add characterStyleOverrides detection` |
| 2 | design-analyst | Empty override bug handling | `fix: handle empty styleOverrideTable entries` |
| 3 | design-analyst | textAutoResize + frame dimensions | `feat: add textAutoResize and frame dimensions` |
| 4 | code-generator-swiftui | Text sizing modifiers | `feat: add text auto-resize handling` |
| 5 | design-validator | Auto Layout property extraction | `feat: add Auto Layout property extraction` |
| 6 | design-analyst | Auto Layout spec output | `feat: add Auto Layout properties to spec` |
| 7 | code-generator-swiftui | Adaptive layout patterns | `feat: add adaptive layout patterns` |
| 8 | compliance-checker | Tablet verification | `feat: add tablet layout verification` |

**Total:** 8 tasks, 4 files modified, 0 files created, 8 commits
