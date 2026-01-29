# Visual Diff Report Template

> **Used by:** compliance-checker

> Reference document for compliance-checker agent. Defines the output format for visual diff reports.

## Report Format

Write to: `docs/figma-reports/{file_key}-visual-diff.md`

```markdown
# Visual Diff Report: {design_name}

**Figma Screenshot:** {screenshot_path}
**Generated Code Files:** {list of files}
**Date:** {timestamp}

## Differences Found

### Diff 1: {Element Name}

| Aspect | Figma (Visual) | Code (Generated) | Severity |
|--------|---------------|-------------------|----------|
| Text Color | "Hook" appears yellow (#F2F20D) | `.foregroundColor(.white)` on entire text | HIGH |
| Text Decoration | "Hook" has underline | No `.underline()` modifier | HIGH |

**Figma Evidence:** In the screenshot, the word "Hook" in the heading is visibly a different color (bright yellow) from the rest of the white text, and has an underline decoration.

**Code Location:** `AIAnalysisView.swift:88-90`

**Suggested Fix:**
```swift
// Replace single Text with concatenation
(Text("Let's fix your ")
    .foregroundColor(.white)
+ Text("Hook")
    .foregroundColor(Color.viralYellow)
    .underline())
    .font(.custom("Poppins-SemiBold", size: 24))
```

---

### Diff 2: {Element Name}
... (same format)
```

## Severity Levels

- **HIGH:** Visually obvious difference (wrong color, missing element, wrong icon)
- **MEDIUM:** Subtle difference (opacity off, spacing slightly wrong)
- **LOW:** Minor difference (font rendering, anti-aliasing)

## Rules

- Always capture at least one Figma screenshot before generating the diff
- Compare EVERY visible element in the screenshot against the code
- Include code file and line number for each difference
- Include a suggested fix for HIGH and MEDIUM severity items
- The diff report is a separate file from the final report
