# Mandatory Spec Creator Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Enforce Design Analyst (Phase 2) execution to create Implementation Spec with explicit layer order documentation, fixing layout accuracy issues.

**Architecture:** Modify figma-to-code skill orchestration to always execute Phase 2, update Design Analyst agent to capture Figma layer hierarchy, modify Code Generator agents to parse and respect layer order from spec.

**Tech Stack:** pb-figma plugin (Claude Code), Pixelbyte Figma MCP server, markdown agents, YAML frontmatter

---

## Background

**Problem:** Integration test revealed page control dots positioned at bottom instead of top. Generated code doesn't match Figma design pixel-perfectly.

**Root cause:** Design Analyst agent (Phase 2) was skipped. Code Generator worked directly from Figma API responses without Implementation Spec, causing layer order information loss.

**Test case:** https://www.figma.com/design/ElHzcNWC8pSYTz2lhPP9h0/ViralZ?node-id=3-29&m=dev
- Expected: Page control dots at TOP (below status bar)
- Actual: Page control dots at BOTTOM (below Continue button)

**Solution:** Make Spec Creator mandatory with explicit layer order documentation.

---

## Task 1: Update figma-to-code Skill - Enforce Phase 2

**Files:**
- Modify: `plugins/pb-figma/skills/figma-to-code/SKILL.md:92-109`

**Step 1: Add Phase 2 enforcement check**

Add after Phase 1 (Design Validator), before Code Generator dispatch:

```markdown
## Phase 2: Mapping & Planning (MANDATORY)

**Agent:** design-analyst
**Purpose:** Create Implementation Spec with explicit layer order

**CRITICAL:** This phase MUST execute. Never skip to Code Generator directly.

**Dispatch:**
```
Task(
  subagent_type="pb-figma:design-analyst",
  prompt=f"Create Implementation Spec for {figma_url} at docs/figma-reports/{{file_key}}-{{node_id}}-spec.md"
)
```

**Validation:**
- Check spec file exists at expected path
- Verify spec contains Layer Order section
- If missing, halt workflow with error message

**Common mistakes:**
- ❌ Skip Phase 2 when project has existing components
- ❌ Generate code directly from Figma API
- ✅ ALWAYS create spec first, even for simple designs
```

**Step 2: Update agent invocation order**

Modify line 92-109 to make Phase 2 explicit:

```markdown
**Agent invocation order:**

1. **Design Validator** → Validates Figma URL, checks access
2. **Design Analyst** → Creates Implementation Spec (MANDATORY)
3. **Asset Manager** → Downloads and processes images/icons
4. **Code Generator** (framework-specific):
   - React: code-generator-react
   - SwiftUI: code-generator-swiftui
   - Vue: code-generator-vue
   - Kotlin: code-generator-kotlin
5. **Compliance Checker** → Validates against spec

**Framework routing happens at step 4:**
```

**Step 3: Test validation logic**

Create test script:

```bash
#!/bin/bash
# Test spec file validation

cd /Users/yusufdemirkoparan/Projects/pixelbyte-agent-workflows

# Test case 1: Spec file exists
mkdir -p docs/figma-reports
touch docs/figma-reports/test-spec.md
echo "## Layer Order" >> docs/figma-reports/test-spec.md

if grep -q "Layer Order" docs/figma-reports/test-spec.md; then
    echo "✅ Test 1 passed: Spec contains Layer Order section"
else
    echo "❌ Test 1 failed: Layer Order section missing"
    exit 1
fi

# Test case 2: Spec file missing
rm docs/figma-reports/test-spec.md

if [ ! -f docs/figma-reports/test-spec.md ]; then
    echo "✅ Test 2 passed: Missing spec detected"
else
    echo "❌ Test 2 failed: Should detect missing spec"
    exit 1
fi

# Cleanup
rm -f docs/figma-reports/test-spec.md
```

Run: `bash test-spec-validation.sh`
Expected: Both tests pass

**Step 4: Commit**

```bash
git add plugins/pb-figma/skills/figma-to-code/SKILL.md
git commit -m "feat(pb-figma): enforce mandatory Spec Creator phase

- Add Phase 2 validation check
- Update agent invocation order
- Prevent skipping to Code Generator directly
- Fixes layout accuracy issue (page control position)"
```

---

## Task 2: Modify Design Analyst Agent - Capture Layer Order

**Files:**
- Modify: `plugins/pb-figma/agents/design-analyst.md`

**Step 1: Add layer order capture to agent instructions**

Add section after "Component Hierarchy" in Implementation Spec template:

```markdown
## Layer Order

**Purpose:** Explicit z-index/layer hierarchy from Figma. Determines rendering order in code.

**Format:**
```yaml
layerOrder:
  - layer: StatusBar
    zIndex: 1000
    position: top
    children: []

  - layer: PageControl
    zIndex: 900
    position: top-below-status
    absoluteY: 60
    children:
      - Dot1
      - Dot2
      - Dot3

  - layer: HeroImage
    zIndex: 500
    position: center
    children: []

  - layer: ContinueButton
    zIndex: 100
    position: bottom
    absoluteY: 800
    children: []
```

**Capture rules:**
1. Higher zIndex = renders on top
2. Use Figma API `y` coordinate for absoluteY
3. Document parent-child relationships
4. Note positioning context (top/center/bottom)
```

**Step 2: Add Figma API query for layer order**

Add tool usage example:

```markdown
### Extracting Layer Order

Use `figma_get_node_details` for each child node to get `absoluteBoundingBox.y`:

```typescript
// Pseudo-code for layer order extraction
const children = fileStructure.children.sort((a, b) => {
  // Sort by Y coordinate (top to bottom)
  return a.absoluteBoundingBox.y - b.absoluteBoundingBox.y;
});

const layerOrder = children.map((child, index) => ({
  layer: child.name,
  zIndex: 1000 - (index * 100), // Top layer = highest zIndex
  position: getPosition(child.absoluteBoundingBox.y),
  absoluteY: child.absoluteBoundingBox.y,
  children: child.children?.map(c => c.name) || []
}));
```

**Critical:** Always query ALL nodes, even if they seem simple. Layer order matters.
```

**Step 3: Update spec template validation**

Add to agent's self-check section:

```markdown
## Self-Check Before Handoff

- [ ] Implementation Spec file created at correct path
- [ ] Component Hierarchy section complete
- [ ] **Layer Order section complete with zIndex values**
- [ ] **absoluteY coordinates documented for all layers**
- [ ] Design Tokens extracted
- [ ] Asset List generated

**If Layer Order is missing or incomplete:** Query Figma API again with `figma_get_node_details` for all children.
```

**Step 4: Test layer order capture**

Create test with mock Figma response:

```bash
# Test that design-analyst captures layer order correctly
# Expected spec should have:
# - PageControl at zIndex 900, absoluteY: 60
# - ContinueButton at zIndex 100, absoluteY: 800
```

**Step 5: Commit**

```bash
git add plugins/pb-figma/agents/design-analyst.md
git commit -m "feat(pb-figma): add layer order capture to design-analyst

- Add Layer Order section to spec template
- Document zIndex and absoluteY extraction
- Add Figma API query examples
- Update self-check validation"
```

---

## Task 3: Modify Code Generator Agents - Parse Layer Order

**Files:**
- Modify: `plugins/pb-figma/agents/code-generator-react.md`
- Modify: `plugins/pb-figma/agents/code-generator-swiftui.md`

### Subtask 3a: Update React Agent

**Step 1: Add layer order parsing instructions**

Add section before "Code Generation" in code-generator-react.md:

```markdown
## Layer Order Parsing

**CRITICAL:** Read Layer Order from Implementation Spec to determine component rendering order.

**React rendering:** Array order = visual order (first element renders first/behind)

**Example spec:**
```yaml
layerOrder:
  - layer: PageControl (zIndex: 900)
  - layer: HeroImage (zIndex: 500)
  - layer: ContinueButton (zIndex: 100)
```

**Generated JSX order:**
```tsx
// Render in zIndex order (highest first)
<div className="container">
  <PageControl /> {/* zIndex 900 - renders on top */}
  <HeroImage />   {/* zIndex 500 - middle */}
  <Button />      {/* zIndex 100 - bottom */}
</div>
```

**Validation:**
1. Parse layerOrder from spec
2. Sort by zIndex (highest first)
3. Render components in sorted order
4. Use absolute positioning if absoluteY is specified
```

**Step 2: Add positioning logic**

```markdown
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

**Position context mapping:**
- `position: top` → `top-0` or `top-[absoluteY]`
- `position: center` → `top-1/2 -translate-y-1/2`
- `position: bottom` → `bottom-0` or `top-[absoluteY]`
```

**Step 3: Commit React changes**

```bash
git add plugins/pb-figma/agents/code-generator-react.md
git commit -m "feat(pb-figma): add layer order parsing to react agent

- Parse layerOrder from Implementation Spec
- Sort components by zIndex
- Apply absolute positioning with Tailwind
- Map position context to CSS classes"
```

### Subtask 3b: Update SwiftUI Agent

**Step 1: Add layer order parsing for SwiftUI**

Add section before "Code Generation" in code-generator-swiftui.md:

```markdown
## Layer Order Parsing

**CRITICAL:** Read Layer Order from Implementation Spec to determine ZStack ordering.

**SwiftUI ZStack:** Last child renders on top (opposite of HTML/React)

**Example spec:**
```yaml
layerOrder:
  - layer: PageControl (zIndex: 900)
  - layer: HeroImage (zIndex: 500)
  - layer: ContinueButton (zIndex: 100)
```

**Generated SwiftUI order:**
```swift
// ZStack renders bottom-to-top (last = on top)
ZStack(alignment: .top) {
    ContinueButton()  // zIndex 100 - first = bottom
    HeroImage()       // zIndex 500 - middle
    PageControl()     // zIndex 900 - last = on top
}
```

**CRITICAL:** Reverse zIndex sort for SwiftUI (lowest zIndex first in code)
```

**Step 2: Add positioning logic for SwiftUI**

```markdown
### Frame Positioning

When spec includes `absoluteY`, use `.position()` or `.offset()`:

```swift
// PageControl at absoluteY: 60
PageControl()
    .position(x: UIScreen.main.bounds.width / 2, y: 60)
    .zIndex(900)

// ContinueButton at absoluteY: 800
Button("Continue") { }
    .position(x: UIScreen.main.bounds.width / 2, y: 800)
    .zIndex(100)
```

**Position context mapping:**
- `position: top` → `.frame(maxHeight: .infinity, alignment: .top)`
- `position: center` → `.frame(maxHeight: .infinity, alignment: .center)`
- `position: bottom` → `.frame(maxHeight: .infinity, alignment: .bottom)`

**Prefer alignment over absolute positioning** when possible (more responsive).
```

**Step 3: Commit SwiftUI changes**

```bash
git add plugins/pb-figma/agents/code-generator-swiftui.md
git commit -m "feat(pb-figma): add layer order parsing to swiftui agent

- Parse layerOrder from Implementation Spec
- Reverse sort for ZStack ordering (last = top)
- Apply position/offset modifiers
- Map position context to frame alignment"
```

---

## Task 4: Add Validation in Compliance Checker

**Files:**
- Modify: `plugins/pb-figma/agents/compliance-checker.md`

**Step 1: Add layer order validation check**

Add section to compliance checker:

```markdown
## Layer Order Validation

**Purpose:** Verify generated code respects layer order from spec.

**Check:**
1. Read layerOrder from Implementation Spec
2. Parse component rendering order from generated code
3. Compare zIndex order matches code order

**Example validation:**

Spec says:
```yaml
layerOrder:
  - PageControl (zIndex: 900)
  - ContinueButton (zIndex: 100)
```

React code must have:
```tsx
<PageControl /> {/* First = top */}
<Button />      {/* Second = bottom */}
```

SwiftUI code must have:
```swift
Button()      {/* First = bottom */}
PageControl() {/* Last = top */}
```

**Validation result:**
- ✅ PASS: Order matches spec
- ❌ FAIL: Order doesn't match spec → request Code Generator fix
```

**Step 2: Add position validation**

```markdown
### Position Validation

Verify components with `position: top` are actually positioned at top:

**React:** Check for `top-0`, `top-[60px]`, or similar Tailwind classes
**SwiftUI:** Check for `.frame(alignment: .top)` or `.position(y: 60)`

**Common mistakes:**
- Component marked `position: top` in spec but has `bottom-0` class
- absoluteY value doesn't match between spec and code
```

**Step 3: Test validation logic**

```bash
# Test compliance checker catches layer order issues
# Mock spec with PageControl zIndex 900, Button zIndex 100
# Mock code with Button before PageControl (wrong order)
# Expected: Compliance checker returns FAIL
```

**Step 4: Commit**

```bash
git add plugins/pb-figma/agents/compliance-checker.md
git commit -m "feat(pb-figma): add layer order validation to compliance checker

- Validate zIndex order matches code rendering order
- Check position context (top/center/bottom)
- Verify absoluteY coordinates
- Framework-specific validation (React vs SwiftUI)"
```

---

## Task 5: Integration Test with Real Figma URL

**Files:**
- Test: Use real project at `/Users/yusufdemirkoparan/Projects/viralZ`

**Step 1: Clean previous test artifacts**

```bash
cd /Users/yusufdemirkoparan/Projects/viralZ

# Remove previous generated code
rm -rf Sources/TestApp/Views/OnboardingView.swift

# Remove previous specs (if any)
rm -rf docs/figma-reports/*

git status
# Expected: Clean working tree or only untracked files
```

**Step 2: Run figma-to-code with test URL**

Start fresh Claude Code session in viralZ project:

```
/pb-figma:figma-to-code https://www.figma.com/design/ElHzcNWC8pSYTz2lhPP9h0/ViralZ?node-id=3-29&m=dev
```

**Expected behavior:**
1. Design Validator validates URL
2. **Design Analyst creates spec at docs/figma-reports/{file_key}-3-29-spec.md**
3. Asset Manager downloads images
4. Code Generator (SwiftUI) generates code
5. Compliance Checker validates

**Step 3: Verify spec file created**

```bash
# Check spec exists
ls -la docs/figma-reports/

# Expected: Should see spec file like ElHzcNWC8pSYTz2lhPP9h0-3-29-spec.md

# Check Layer Order section exists
grep -A 20 "## Layer Order" docs/figma-reports/*.md

# Expected: Should see layerOrder with PageControl at high zIndex
```

**Step 4: Verify generated code has correct layer order**

```bash
# Check generated SwiftUI code
cat Sources/TestApp/Views/OnboardingView.swift

# Expected for ZStack:
# - PageControl should be LAST in ZStack (renders on top)
# - ContinueButton should be EARLIER in ZStack (renders below)
```

**Step 5: Visual validation**

Build and run in Xcode simulator:

```bash
cd /Users/yusufdemirkoparan/Projects/viralZ
open viralZ.xcodeproj

# Build and run (Cmd+R)
# Expected: Page control dots appear at TOP of screen, not bottom
```

**Step 6: Compare with Figma design**

Open Figma URL side-by-side with simulator:
- Figma: https://www.figma.com/design/ElHzcNWC8pSYTz2lhPP9h0/ViralZ?node-id=3-29&m=dev
- Simulator: iPhone 17 Pro Max

**Validation checklist:**
- [ ] Page control dots at TOP (below status bar) ✅
- [ ] Hero image in CENTER ✅
- [ ] Continue button at BOTTOM ✅
- [ ] Spacing matches Figma ✅
- [ ] Colors match design tokens ✅

**Step 7: Document test results**

```bash
# Create test results document
cat > docs/plans/2026-01-26-mandatory-spec-test-results.md << 'EOF'
# Mandatory Spec Creator Test Results

**Date:** 2026-01-26
**Test URL:** https://www.figma.com/design/ElHzcNWC8pSYTz2lhPP9h0/ViralZ?node-id=3-29&m=dev
**Framework:** SwiftUI

## Test Results

### Phase 2 Execution

✅ Design Analyst created spec at docs/figma-reports/ElHzcNWC8pSYTz2lhPP9h0-3-29-spec.md
✅ Spec contains Layer Order section
✅ zIndex values documented for all layers
✅ absoluteY coordinates captured

### Generated Code

✅ PageControl positioned at TOP (last in ZStack)
✅ ContinueButton positioned at BOTTOM (first in ZStack)
✅ Layer order matches spec

### Visual Validation

✅ Page control dots appear at top of screen
✅ Matches Figma design pixel-perfectly

## Conclusion

Layout accuracy issue FIXED. Mandatory Spec Creator ensures layer order preservation.
EOF

git add docs/plans/2026-01-26-mandatory-spec-test-results.md
```

**Step 8: Commit integration test**

```bash
git add .
git commit -m "test(pb-figma): verify mandatory spec creator fixes layout accuracy

- Phase 2 (Design Analyst) executed successfully
- Implementation Spec contains layer order
- Generated SwiftUI code respects zIndex
- Page control now at TOP (pixel-perfect match with Figma)

Fixes: Page control position issue from integration test"
```

---

## Success Criteria

**Spec Creator enforcement:**
- [ ] figma-to-code skill enforces Phase 2 execution
- [ ] Design Analyst always creates Implementation Spec
- [ ] Spec validation halts workflow if missing

**Layer order capture:**
- [ ] Design Analyst queries Figma API for all node coordinates
- [ ] Spec documents zIndex and absoluteY for all layers
- [ ] Parent-child relationships captured

**Code generation:**
- [ ] React agent sorts components by zIndex (highest first)
- [ ] SwiftUI agent reverse-sorts for ZStack (lowest first)
- [ ] Absolute positioning applied when absoluteY specified

**Validation:**
- [ ] Compliance Checker verifies layer order matches spec
- [ ] Position context (top/center/bottom) validated
- [ ] Integration test with real Figma URL passes

**Visual accuracy:**
- [ ] Page control dots appear at TOP of screen
- [ ] Generated code matches Figma design pixel-perfectly
- [ ] No layout accuracy issues remain

---

## Rollback Plan

If mandatory Spec Creator causes issues:

1. **Revert skill changes:**
```bash
git revert HEAD~5..HEAD  # Revert last 5 commits
```

2. **Make Phase 2 optional but recommended:**
   - Add flag to skip spec creation
   - Log warning when skipped
   - Document trade-offs in skill

3. **Hybrid approach:**
   - Mandatory for complex designs (10+ components)
   - Optional for simple designs (< 10 components)
   - User can force with `--with-spec` flag

---

## Future Enhancements

**Post-v1.1.0 improvements:**

1. **Visual diff validation:**
   - Take screenshot of generated code
   - Compare with Figma screenshot using Claude Vision
   - Auto-detect layout mismatches

2. **Layer order auto-correction:**
   - If Compliance Checker finds mismatch
   - Auto-fix component order in code
   - Re-validate until pass

3. **Responsive design support:**
   - Capture breakpoint-specific layer orders
   - Generate media queries (React) or size classes (SwiftUI)
   - Validate across multiple screen sizes

4. **Animation timeline:**
   - Capture Figma Smart Animate transitions
   - Document animation timing in spec
   - Generate transition code

---

## References

- Original issue: docs/plans/2026-01-25-swiftui-test-results.md
- Brainstorming session: Summary section 8 (layout accuracy investigation)
- figma-to-code skill: plugins/pb-figma/skills/figma-to-code/SKILL.md
- Test Figma URL: https://www.figma.com/design/ElHzcNWC8pSYTz2lhPP9h0/ViralZ?node-id=3-29&m=dev
