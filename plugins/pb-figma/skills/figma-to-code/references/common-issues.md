# Common Issues and Solutions

> **Used by:** code-generator-*

Common problems and solutions encountered in Figma-to-code conversion.

## Typography Issues

### Issue: Line-Height Mismatch

**Symptom:** Line spacing between text is different from Figma

**Cause:** Figma and CSS calculate line-height differently
- Figma: Places extra space above the next line
- CSS: Distributes space equally above and below (half-leading)

**Solution:**
```jsx
// WRONG
<p className="leading-[150%]">

// CORRECT
<p className="leading-[1.5]">
```

**Formula:** `CSS line-height = Figma percentage ÷ 100`

---

### Issue: Letter-Spacing Wrong

**Symptom:** Character spacing doesn't match

**Cause:** Figma tracking value uses different units

**Solution:**
```jsx
// Figma tracking: -20

// WRONG
<p className="tracking-[-20px]">

// CORRECT
<p className="tracking-[-0.02em]">
```

**Formula:** `CSS em = Figma tracking ÷ 1000`

---

### Issue: Font Renders Differently

**Symptom:** Same font appears with different weight

**Solutions:**

1. Add font smoothing:
```css
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
```

2. Verify font file is loaded correctly:
```jsx
// Is font definition correct in globals.css or tailwind.config.js?
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
```

3. Check font-weight matching:
```
Figma "Semi Bold" → font-semibold (600)
Figma "Medium" → font-medium (500)
```

## Layout Issues

### Issue: Div Soup

**Symptom:** Everything made with `<div>`, no accessibility

**Cause:** AI didn't consider semantic meaning

**Solution:** Use semantic HTML

```jsx
// WRONG
<div onClick={handleClick}>Click me</div>
<div>
  <div>Item 1</div>
  <div>Item 2</div>
</div>

// CORRECT
<button onClick={handleClick}>Click me</button>
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
</ul>
```

**Reference table:**

| Element type | Semantic HTML |
|--------------|---------------|
| Clickable action | `<button>` |
| Link/navigation | `<a href>` |
| List | `<ul>` + `<li>` |
| Ordered list | `<ol>` + `<li>` |
| Navigation | `<nav>` |
| Header | `<header>` |
| Footer | `<footer>` |
| Main content | `<main>` |
| Section | `<section>` |
| Form | `<form>` |

---

### Issue: Flexbox Direction Wrong

**Symptom:** Elements horizontal instead of vertical (or vice versa)

**Cause:** Auto Layout direction converted incorrectly

**Solution:**
```jsx
// Figma: Horizontal Auto Layout
<div className="flex flex-row">

// Figma: Vertical Auto Layout
<div className="flex flex-col">
```

---

### Issue: Gap/Spacing Inconsistent

**Symptom:** Space between elements different from Figma

**Solutions:**

1. Convert gap value correctly:
```jsx
// Figma gap: 16px
<div className="flex gap-4">  // ✓ 4 × 4 = 16px
```

2. Don't mix padding and margin:
```jsx
// Container padding separate, element gap separate
<div className="p-6 flex gap-4">
```

3. Use arbitrary value if not in Tailwind scale:
```jsx
// 18px gap → not in scale
<div className="gap-[18px]">
```

---

### Issue: Fill Container Not Working

**Symptom:** Element doesn't fill container

**Solution:**
```jsx
// Parent must be flex
<div className="flex">
  {/* flex-1 for fill container */}
  <div className="flex-1">This fills</div>
  <div className="w-fit">This is hug</div>
</div>
```

## Color Issues

### Issue: Opacity Not Applied

**Symptom:** Colors darker/lighter than expected

**Solution:**
```jsx
// Figma: #FF5733 at 80% opacity

// WRONG
<div className="bg-[#FF5733]">

// CORRECT
<div className="bg-[#FF5733]/80">

// OR
<div className="bg-[rgba(255,87,51,0.8)]">
```

---

### Issue: Gradient Wrong

**Symptom:** Gradient direction or colors different

**Solution:**
```jsx
// Figma: Linear gradient 135°, #FF5733 to #33FF57

// Tailwind gradient directions:
// to-r: 90° (right)
// to-br: 135° (bottom right)
// to-b: 180° (down)

<div className="bg-gradient-to-br from-[#FF5733] to-[#33FF57]">

// For custom angle:
<div className="bg-[linear-gradient(135deg,#FF5733,#33FF57)]">
```

## Responsive Issues

### Issue: No Responsive Breakpoints

**Symptom:** Design only works at one size

**Solution:** Add mobile-first responsive

```jsx
// Mobile: single column, Desktop: two columns
<div className="flex flex-col md:flex-row gap-4">
  <div className="w-full md:w-1/2">Left</div>
  <div className="w-full md:w-1/2">Right</div>
</div>

// Mobile: hide, Desktop: show
<div className="hidden lg:block">Only visible on desktop</div>
```

**Breakpoint reference:**
```
sm:  640px and up
md:  768px and up
lg:  1024px and up
xl:  1280px and up
2xl: 1536px and up
```

---

### Issue: Fixed Width Not Responsive

**Symptom:** Component overflows on mobile

**Solution:**
```jsx
// WRONG - fixed width
<div className="w-[400px]">

// CORRECT - with max-width
<div className="w-full max-w-[400px]">

// OR responsive width
<div className="w-full sm:w-[400px]">
```

## Accessibility Issues

### Issue: No Focus State

**Symptom:** Can't tell where you are when tabbing

**Solution:**
```jsx
<button className="focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
  Click me
</button>
```

---

### Issue: Alt Text Missing

**Symptom:** Images have no description

**Solution:**
```jsx
// Decorative image
<img src="..." alt="" aria-hidden="true" />

// Meaningful image
<img src="..." alt="Product photo: Blue t-shirt" />

// Icon button
<button aria-label="Open menu">
  <MenuIcon />
</button>
```

---

### Issue: Insufficient Color Contrast

**Symptom:** Text hard to read

**Check:** WCAG AA standard = 4.5:1 contrast ratio

**Solution:** Use darker/lighter color combination
```jsx
// WRONG - low contrast
<p className="text-gray-400 bg-gray-200">

// CORRECT - sufficient contrast
<p className="text-gray-700 bg-gray-100">
```

## MCP and API Issues

### Issue: Rate Limit Error

**Symptom:** Figma API returns 429

**Solutions:**

1. Cache requests
2. Don't make unnecessary requests
3. Wait and retry (exponential backoff)

```javascript
// Exponential backoff
async function fetchWithRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await sleep(Math.pow(2, i) * 1000); // 1s, 2s, 4s
        continue;
      }
      throw error;
    }
  }
}
```

---

### Issue: Node ID Not Found

**Symptom:** "Node not found" error

**Solution:**
```
URL: figma.com/design/abc123/MyDesign?node-id=1-2

Correct format:
fileKey: abc123
nodeId: 1:2 (colon not hyphen!)
```

---

### Issue: Large Design Exceeds Context Limit

**Symptom:** Response truncated or throws error

**Solutions:**

1. Get structure first with `get_metadata`
2. Process only needed nodes
3. Split design into parts

## Phase 4: Visual Validation Issues

### Issue: Full Page Screenshot Taken

**Symptom:** Figma frame and browser screenshot sizes don't match

**Solution:** Zoom to element or use scroll_to:
```javascript
// 1. Get tab context
mcp__claude-in-chrome__tabs_context_mcp({
  createIfEmpty: true
})

// 2. Get accessibility tree (to find element ref)
mcp__claude-in-chrome__read_page({
  tabId: <tab-id>
})

// 3. Scroll to element
mcp__claude-in-chrome__computer({
  action: "scroll_to",
  ref: "ref_1",  // ref from read_page
  tabId: <tab-id>
})

// 4. Zoom to specific region
mcp__claude-in-chrome__computer({
  action: "zoom",
  region: [x0, y0, x1, y1],  // Element coordinates
  tabId: <tab-id>
})
```

---

### Issue: Element Ref Not Found

**Symptom:** Component not visible in `read_page()` output

**Solutions:**

1. Add `data-testid` to component:
```jsx
<div data-testid="hero-card">...</div>
```

2. Wait for page to load (take snapshot after 2-3 seconds)

3. Make sure you're on the correct page

---

### Issue: Claude Vision Not Catching Differences

**Symptom:** Obvious differences not detected

**Solutions:**

1. Take higher resolution screenshot (`scale: 2`)
2. Ask for specific comparison categories:
   - Typography: font-size, weight, color
   - Spacing: padding, margin, gap
   - Colors: background, border
3. Mark critical areas

---

### Issue: Still Differences After 3 Iterations

**Symptom:** Some differences won't fix

**Solution:**
- Accept minor differences (1-2px spacing, font rendering)
- Notify user: "Minor differences exist, manual review recommended"
- Proceed to Phase 5

## Quick Fix Reference

| Issue | Quick Fix |
|-------|-----------|
| Line-height wrong | Use decimal instead of `%` |
| Tracking wrong | Convert to em with `÷1000` |
| Div soup | Use semantic HTML |
| Gap not matching | Check Tailwind scale |
| No opacity | Use `/80` syntax |
| No responsive | Add `md:` prefix |
| No focus | Add `focus:ring-2` |
| No alt text | Write meaningful `alt` |
| Full page screenshot | Use `element` + `ref` parameters |
| No element ref | Add `data-testid` |
| Wrong Node ID | Convert hyphen `-` → colon `:` |
