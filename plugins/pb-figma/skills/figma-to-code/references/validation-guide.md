# Validation Guide

> **Used by:** design-validator

Claude Vision-based strategy for verifying that generated code matches the Figma design.

## Goal

Detect and fix visual differences between Figma design and implementation.

---

## Validation Workflow

```
1. Take Figma Screenshot
   └─→ Pixelbyte Figma MCP

2. Take Browser Screenshot
   └─→ Claude in Chrome MCP

3. Claude Vision Comparison
   └─→ Categorize differences

4. List with TodoWrite
   └─→ One todo per difference

5. Fix and Re-check
   └─→ Complete todos
```

---

## Check Categories

### 1. Typography

| Check | What to Look For |
|-------|------------------|
| Font family | Same as Figma? |
| Font size | Pixel value matches? |
| Font weight | Bold, medium, regular correct? |
| Line height | Line spacing same? |
| Letter spacing | Character spacing present? |
| Text color | Color code matches? |

**Common Mistakes:**
- Using `text-sm` instead of `text-base`
- Using `font-medium` instead of `font-bold`
- Forgetting `leading-tight`

### 2. Spacing

| Check | What to Look For |
|-------|------------------|
| Padding | Inner spacing equal? |
| Margin | Outer spacing equal? |
| Gap | Space between elements correct? |

**Common Mistakes:**
- Using `p-2` instead of `p-4`
- Using `gap-2` instead of `gap-4`
- Forgetting asymmetric padding (`pt-6 pb-4`)

### 3. Colors

| Check | What to Look For |
|-------|------------------|
| Background | Background color correct? |
| Border | Border color correct? |
| Text | Text color correct? |
| Shadow | Shadow color/opacity correct? |

**Common Mistakes:**
- Not using Tailwind colors instead of hardcoded hex
- Forgetting opacity values
- Not using custom colors from `globals.css`
  - e.g., Use `bg-orange-1` instead of `#FE4601`
  - e.g., Use `bg-grey-1` instead of `#1A1A1A`

### 4. Layout

| Check | What to Look For |
|-------|------------------|
| Flex direction | row/column correct? |
| Alignment | items-center, justify-between correct? |
| Width/Height | Dimensions match? |
| Position | Relative/absolute correct? |

**Common Mistakes:**
- Using `flex-row` instead of `flex-col`
- Forgetting `items-center`
- Missing responsive breakpoints

### 5. Assets

| Check | What to Look For |
|-------|------------------|
| Icons | Size and color correct? |
| Images | Aspect ratio preserved? |
| Border radius | Corner rounding equal? |
| Shadows | Box shadow values correct? |

**Common Mistakes:**
- Icon size being `size-5` instead of `size-4`
- Using `rounded-md` instead of `rounded-lg`
- Different shadow intensity

---

## Claude Vision Analysis Format

Use this format when comparing two visuals:

```markdown
## 🔍 Figma vs Implementation Comparison

### Typography
| Element | Figma | Implementation | Status | Fix |
|---------|-------|----------------|--------|-----|
| Title | 32px bold | 28px medium | ❌ | `text-3xl font-bold` |
| Subtitle | 16px gray-500 | 16px gray-500 | ✅ | - |

### Spacing
| Element | Figma | Implementation | Status | Fix |
|---------|-------|----------------|--------|-----|
| Card padding | 24px | 16px | ❌ | `p-6` |
| Button gap | 8px | 8px | ✅ | - |

### Colors
| Element | Figma | Implementation | Status | Fix |
|---------|-------|----------------|--------|-----|
| Primary | #FE4601 | #3B82F6 | ❌ | `bg-orange-1` |
| Background | #FFFFFF | #FFFFFF | ✅ | - |

### Layout
| Element | Figma | Implementation | Status | Fix |
|---------|-------|----------------|--------|-----|
| Direction | row | row | ✅ | - |
| Alignment | center | center | ✅ | - |

### Assets
| Element | Figma | Implementation | Status | Fix |
|---------|-------|----------------|--------|-----|
| Icon size | 24px | 20px | ❌ | `size-6` |
| Border radius | 16px | 12px | ❌ | `rounded-2xl` |

---

## Summary
- ✅ Correct: 5 items
- ❌ To fix: 4 items

## Todo List
1. Title font-size: text-2xl → text-3xl
2. Card padding: p-4 → p-6
3. Button background: bg-blue-500 → bg-orange-1
4. Icon size: size-5 → size-6
```

---

## Quick Checklist

Before validation:
```
□ Correct frame selected in Figma?
□ Dev server running? (localhost:3000)
□ Component rendering on correct page?
```

During validation:
```
□ Typography checked
□ Spacing checked
□ Colors checked
□ Layout checked
□ Assets checked
```

After validation:
```
□ All differences listed with TodoWrite
□ Todos completed in order
□ Final check done
□ If no differences → proceed to Phase 5
```

---

## Tool Reference

### Figma Screenshot

```javascript
mcp__pixelbyte-figma-mcp__figma_get_screenshot({
  params: {
    file_key: "FIGMA_FILE_KEY",
    node_ids: ["NODE_ID"],
    format: "png",
    scale: 2
  }
})
```

### Browser Screenshot (Claude in Chrome MCP)

**⚠️ Tab context must be obtained first!**

```javascript
// 1. Get tab context (ALWAYS FIRST STEP!)
mcp__claude-in-chrome__tabs_context_mcp({
  createIfEmpty: true
})
// → Save the returned tabId

// 2. Navigate to page
mcp__claude-in-chrome__navigate({
  url: "http://localhost:3000/[component-path]",
  tabId: <tab-id>
})

// 3. Wait and get accessibility tree (to find element ref)
mcp__claude-in-chrome__computer({
  action: "wait",
  duration: 2,
  tabId: <tab-id>
})

mcp__claude-in-chrome__read_page({
  tabId: <tab-id>
})
// → Returns element refs in ref="ref_1" format

// 4. Take screenshot
mcp__claude-in-chrome__computer({
  action: "screenshot",
  tabId: <tab-id>
})

// 5. (Optional) Zoom to specific region
mcp__claude-in-chrome__computer({
  action: "zoom",
  region: [x0, y0, x1, y1],
  tabId: <tab-id>
})
```

**Best Practice:** Add `data-testid` to components:
```tsx
<div data-testid="hero-card">...</div>
```

**Finding elements:** You can find elements using natural language:
```javascript
mcp__claude-in-chrome__find({
  query: "hero card component",
  tabId: <tab-id>
})
```

---

## Detailed Instructions

For step-by-step validation process, see: `visual-validation-loop.md`
