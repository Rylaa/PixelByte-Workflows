# Visual Validation Loop - Claude Vision Approach

This document explains the **Claude Vision-based simple approach** for Phase 4: Visual Validation.

## Core Principle

```
Figma Screenshot + Browser Screenshot → Claude Vision Comparison → TodoWrite
```

Instead of complex tools (ImageMagick, RMSE calculation), we use Claude's visual analysis capabilities to detect differences.

---

## Workflow Overview

```
┌──────────────────────────────────────────────────────────────┐
│                    VISUAL VALIDATION LOOP                     │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. FIGMA SCREENSHOT                                          │
│     └─→ Pixelbyte Figma MCP: figma_get_screenshot            │
│                                                               │
│  2. BROWSER SCREENSHOT                                        │
│     └─→ Claude in Chrome MCP: computer({action: "screenshot"})│
│                                                               │
│  3. CLAUDE VISION COMPARISON                                  │
│     └─→ Analyze both images, list differences                │
│                                                               │
│  4. DIFFERENCE LIST WITH TODOWRITE                           │
│     └─→ One todo item for each difference                    │
│                                                               │
│  5. FIX AND RE-CHECK                                         │
│     └─→ Complete todos, take new screenshot if needed        │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Step 1: Take Figma Screenshot

**Use Pixelbyte Figma MCP:**

```javascript
mcp__pixelbyte-figma-mcp__figma_get_screenshot({
  params: {
    file_key: "FIGMA_FILE_KEY",  // From URL: figma.com/design/FILE_KEY/...
    node_ids: ["NODE_ID"],        // Node ID in 123:456 format
    format: "png",
    scale: 2                      // For 2x quality
  }
})
```

**Response:** Returns Figma CDN URL (e.g., `https://figma-alpha-api.s3.us-west-2.amazonaws.com/...`)

To view this URL with Claude Vision:
- Share URL directly in message → Claude views automatically
- Or fetch content with `WebFetch`

---

## Step 2: Take Browser Screenshot (Claude in Chrome MCP)

**⚠️ IMPORTANT:** Tab context MUST be obtained first!

**Screenshot with Claude in Chrome MCP:**

```javascript
// 1. Get tab context (ALWAYS FIRST STEP!)
mcp__claude-in-chrome__tabs_context_mcp({
  createIfEmpty: true
})
// → Save returned tabId and use for all subsequent operations

// 2. Navigate to dev server
mcp__claude-in-chrome__navigate({
  url: "http://localhost:3000/[component-path]",
  tabId: <returned-tab-id>
})

// 3. Wait for page to load
mcp__claude-in-chrome__computer({
  action: "wait",
  duration: 2,
  tabId: <tab-id>
})
```

**Get Accessibility Tree (to find element ref):**
```javascript
// 4. Get page accessibility tree
mcp__claude-in-chrome__read_page({
  tabId: <tab-id>
})
```

**read_page output example:**
```
- main
  - div "container"
    - article ref="ref_1" "HeroCard"    ← Use this ref
      - h2 "Title"
      - p "Description"
    - button ref="ref_2" "Submit"
```

**Take Screenshot:**
```javascript
// 5. Take full page screenshot
mcp__claude-in-chrome__computer({
  action: "screenshot",
  tabId: <tab-id>
})
// → Returns imageId, viewable by Claude Vision
```

**Focus on Specific Element:**
```javascript
// Scroll element into view
mcp__claude-in-chrome__computer({
  action: "scroll_to",
  ref: "ref_1",  // ref from read_page
  tabId: <tab-id>
})

// Zoom to specific region
mcp__claude-in-chrome__computer({
  action: "zoom",
  region: [100, 200, 500, 600],  // [x0, y0, x1, y1] coordinates
  tabId: <tab-id>
})
```

**Find Element with Natural Language:**
```javascript
// Find element using natural language
mcp__claude-in-chrome__find({
  query: "hero card component",
  tabId: <tab-id>
})
// → Returns refs of matching elements
```

**Tips:**
- If dev server not running → Run `npm run dev` with Bash
- If element ref not found → Check `read_page` output or use `find`
- Adding data-testid → Recommended for reliable element finding
- Full page screenshot → Take after making component visible

---

## Step 3: Compare with Claude Vision

Compare both images side by side to detect differences.

### Check Categories

| Category | Items to Check |
|----------|----------------|
| **Typography** | Font family, size, weight, line-height, letter-spacing, color |
| **Spacing** | Padding (all directions), margin, gap |
| **Colors** | Background, border, text, shadow colors |
| **Layout** | Flex direction, alignment, justify, width, height |
| **Assets** | Icon size/color, image aspect ratio, border-radius |

### Example Analysis Output

```markdown
## 🔍 Figma vs Implementation Comparison

### ❌ Typography Differences
| Element | Figma | Implementation | Tailwind Fix |
|---------|-------|----------------|--------------|
| Title | 32px bold | 28px medium | `text-3xl font-bold` |
| Description | 16px gray-500 | 14px gray-400 | `text-base text-gray-500` |

### ❌ Spacing Differences
| Element | Figma | Implementation | Tailwind Fix |
|---------|-------|----------------|--------------|
| Card padding | 24px | 16px | `p-6` |
| Button gap | 12px | 8px | `gap-3` |

### ❌ Color Differences
| Element | Figma | Implementation | Tailwind Fix |
|---------|-------|----------------|--------------|
| Primary button | #FE4601 | #3B82F6 | `bg-orange-1` |

### ✅ Layout
Correct - flexbox direction and alignment match.

### ✅ Assets
Correct - icon sizes and border-radius match.
```

---

## Step 4: Create Difference List with TodoWrite

Create a todo item for each detected difference:

```javascript
TodoWrite({
  todos: [
    {
      content: "Title font-size: text-2xl → text-3xl",
      status: "pending",
      activeForm: "Fixing title font-size"
    },
    {
      content: "Card padding: p-4 → p-6",
      status: "pending",
      activeForm: "Fixing card padding"
    },
    {
      content: "Button background: bg-blue-500 → bg-orange-1",
      status: "pending",
      activeForm: "Fixing button background"
    },
    {
      content: "Description text color: text-gray-400 → text-gray-500",
      status: "pending",
      activeForm: "Fixing description text color"
    }
  ]
})
```

### Recommended Todo Format

```
[Element] [Property]: [Current Value] → [Correct Value]
```

Examples:
- `Title font-size: text-2xl → text-3xl`
- `Card padding: p-4 → p-6`
- `Icon color: text-gray-500 → text-white`
- `Gap between buttons: gap-2 → gap-4`

---

## Step 5: Fix and Re-check

### Fix Process

1. **Set todo to in_progress**
2. **Fix the code** (using Edit tool)
3. **Mark todo as completed**
4. **Move to next todo**

### Re-check

When all todos are complete:

1. Take new browser screenshot
2. Compare with Figma screenshot again
3. If new differences found → Add new todo
4. If no differences → Proceed to Phase 5

**⚠️ Max 3 iterations:** After 3 checks, if differences remain, notify user and proceed to Phase 5. Some differences (font rendering, subpixel) may be unfixable.

---

## Quick Reference

### Claude in Chrome MCP Tools

| Tool | Usage |
|------|-------|
| `tabs_context_mcp` | Get tab context (FIRST STEP!) |
| `tabs_create_mcp` | Create new tab |
| `navigate` | Navigate to URL |
| `computer({action: "screenshot"})` | Take full page screenshot |
| `computer({action: "zoom"})` | Zoom to specific region |
| `computer({action: "scroll_to"})` | Scroll to element |
| `computer({action: "wait"})` | Wait for specified duration |
| `read_page` | Get accessibility tree |
| `find` | Find element with natural language |
| `javascript_tool` | Execute JS on page |

### Pixelbyte Figma MCP Tools

| Tool | Usage |
|------|-------|
| `figma_get_screenshot` | Take node screenshot |
| `figma_get_node_details` | Get node details |
| `figma_get_design_tokens` | Get design tokens |

---

## Responsive Test (Optional)

To test at different viewport sizes:

```javascript
// Change window size
mcp__claude-in-chrome__resize_window({
  width: 375,   // Mobile
  height: 812,
  tabId: <tab-id>
})

// Take screenshot
mcp__claude-in-chrome__computer({
  action: "screenshot",
  tabId: <tab-id>
})

// Return to desktop
mcp__claude-in-chrome__resize_window({
  width: 1440,
  height: 900,
  tabId: <tab-id>
})
```

---

## Console Log Debugging

Read console logs for debugging:

```javascript
mcp__claude-in-chrome__read_console_messages({
  tabId: <tab-id>,
  pattern: "error|warning",  // Filter only errors and warnings
  onlyErrors: true           // Or get only errors
})
```

---

## Checklist

```
□ Tab context obtained (tabs_context_mcp)
□ Figma screenshot taken
□ Browser screenshot taken
□ Claude Vision comparison done
□ Differences listed with TodoWrite
□ All todos completed
□ Final check performed
□ No differences → Proceed to Phase 5
```
