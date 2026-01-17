# Phase 4: Visual Validation (Claude Vision)

This prompt detects differences by comparing Figma screenshot and Browser screenshot with Claude Vision.

## Approach

```
Figma Screenshot + Browser Screenshot → Claude Vision Comparison → TodoWrite
```

Simple and effective. NO pixel-based percentage calculation. We use Claude Vision's visual analysis capabilities.

---

## Workflow

```
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: Take Figma screenshot                               │
│         → Pixelbyte MCP: figma_get_screenshot              │
├─────────────────────────────────────────────────────────────┤
│ STEP 2: Take Browser screenshot (Claude in Chrome)          │
│         → tabs_context_mcp() → get tabId                   │
│         → navigate() → go to page                          │
│         → computer({action: "screenshot"})                 │
├─────────────────────────────────────────────────────────────┤
│ STEP 3: Compare with Claude Vision                          │
│         → Examine both visuals side by side                │
│         → Categorize differences                           │
├─────────────────────────────────────────────────────────────┤
│ STEP 4: Create difference list with TodoWrite              │
│         → One todo item for each difference                │
├─────────────────────────────────────────────────────────────┤
│ STEP 5: Fix and re-check                                    │
│         → Complete todos                                   │
│         → Max 3 iterations                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Claude Vision Comparison Prompt

```markdown
Compare both visuals and list differences.

**Visual 1: Figma Design (Target)**
[Figma screenshot]

**Visual 2: Code Output (Current)**
[Browser screenshot]

**Check Categories:**

1. TYPOGRAPHY
   - Font family, size, weight
   - Line height, letter spacing
   - Text color

2. SPACING
   - Padding (top, right, bottom, left)
   - Margin
   - Gap between elements

3. COLORS
   - Background colors
   - Border colors
   - Text colors
   - Shadow colors

4. LAYOUT
   - Element alignment
   - Flex direction
   - Width/height

5. ASSETS
   - Icons (size, color)
   - Images (aspect ratio)
   - Border radius

**Output Format:**

For each difference:
| Element | Figma | Implementation | Tailwind Fix |
|---------|-------|----------------|--------------|
| Title | 32px bold | 28px medium | `text-3xl font-bold` |

**Result:**
- Are there critical differences?
- What are the minor differences?
- Is correction needed?
```

---

## Example Analysis Output

```markdown
## 🔍 Figma vs Implementation Comparison

### Typography
| Element | Figma | Implementation | Fix |
|---------|-------|----------------|-----|
| Title | 32px bold | 28px medium | `text-3xl font-bold` |
| Description | 16px gray-500 | 14px gray-400 | `text-base text-gray-500` |

### Spacing
| Element | Figma | Implementation | Fix |
|---------|-------|----------------|-----|
| Card padding | 24px | 16px | `p-6` |
| Button gap | 12px | 8px | `gap-3` |

### Colors
| Element | Figma | Implementation | Fix |
|---------|-------|----------------|-----|
| Primary button | #FE4601 | #3B82F6 | `bg-orange-1` |

### Layout
✅ Correct - flexbox direction and alignment match.

### Assets
| Element | Figma | Implementation | Fix |
|---------|-------|----------------|-----|
| Border radius | 16px | 12px | `rounded-2xl` |

---

## Result
- ❌ Critical: 1 (primary button color wrong)
- ⚠️ Minor: 4 (typography, spacing, radius)
- ✅ Correction required
```

---

## TodoWrite Format

For each detected difference:

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
    }
  ]
})
```

---

## Decision Criteria

| Situation | Action |
|-----------|--------|
| No critical differences | → Proceed to Phase 5 |
| Minor differences exist | → Fix, re-check |
| Critical difference exists | → Fix immediately, re-check |
| After 3 iterations | → Notify user, proceed to Phase 5 |

**Critical Difference Examples:**
- Primary color completely wrong
- Layout direction wrong (row vs col)
- Element missing

**Minor Difference Examples:**
- 1-2px spacing difference
- Slight font-weight difference
- Small border radius difference

---

## Iteration Tracking

```
Iteration 1:
├── Take screenshot
├── Compare with Claude Vision
├── Differences: [padding, font-weight, color]
├── List with TodoWrite
├── Fix
└── Result: Critical difference fixed, minors remain → Continue

Iteration 2:
├── Take screenshot
├── Compare with Claude Vision
├── Differences: [small spacing]
├── Fix
└── Result: Minor differences fixed → Continue

Iteration 3:
├── Take screenshot
├── Compare with Claude Vision
├── Differences: [none or acceptable]
└── Result: ✅ Complete → Proceed to Phase 5
```

---

## Usage

1. Take Figma screenshot (Pixelbyte MCP)
2. Take Browser screenshot (Claude in Chrome MCP)
3. Analyze with Claude Vision using this prompt
4. List differences with TodoWrite
5. Fix and re-test (max 3 iterations)
