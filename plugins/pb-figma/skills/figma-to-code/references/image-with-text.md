# Image-with-Text Handling Reference

Comprehensive reference for detecting and handling illustrations that contain embedded text labels, preventing duplicate text generation during Figma-to-code conversion.

## Problem Statement

Some illustration assets (charts, badges, diagrams) already contain text labels baked into the image. When these assets are exported as images and the code generator also creates separate `Text()` / `<span>` elements for those same labels, the result is visual duplication -- the text appears twice.

**Common scenarios:**

| Scenario | Example | Risk |
|----------|---------|------|
| Chart axis labels | "PROJECTED GROWTH" text in a bar chart | Text duplicated above/below chart image |
| Badge text | "PRO" or "NEW" label inside a badge illustration | Label rendered twice |
| Watermark text | Brand name overlaid on a decorative image | Watermark shown as both image content and text node |
| Infographic labels | Data callouts embedded in an infographic | Numbers/labels duplicated |

---

## Detection Approaches

There are two complementary detection paths. Both converge on the same outcome: suppress the duplicate text node and use accessibility labels instead.

### Path 1: `[contains-text]` Annotation (Explicit)

The design-analyst agent adds a `[contains-text]` annotation to the Asset Children entry when it detects that a `DOWNLOAD_AS_IMAGE` node has text children.

**Suppression process (design-analyst):**

1. Parse "Flagged for LLM Review" section for `DOWNLOAD_AS_IMAGE` entries
2. For each flagged node ID, query `figma_get_node_details` to find text children
3. Collect text child node IDs into a suppression list
4. When generating component tables, skip any node whose ID is in the suppression list
5. Add a `[contains-text]` note to the Asset Children entry

**Spec output:**

```markdown
| Property | Value |
|----------|-------|
| **Asset Children** | `IMAGE:growth-chart:6:32:354:132 [contains-text: "PROJECTED GROWTH"]` |
```

**Code-generator signal:**
- Do NOT generate `Text("PROJECTED GROWTH")` as a sibling
- The image already contains this text visually
- Use the text content for `accessibilityLabel` instead

### Path 2: Flagged Frame Heuristic (Indirect)

When no explicit annotation exists, the code generator can detect image-with-text from the "Flagged for LLM Review" section.

**Detection rules:**

1. A flagged frame was decided as `DOWNLOAD_AS_IMAGE` by asset-manager
2. AND the frame has a text-like name (contains capitalized words)
3. The image likely contains that text

**Detection algorithm:**

```
For each flagged illustration asset:
1. Check if asset name contains text words (PROJECTED GROWTH, TITLE, LABEL)
2. Check if parent component in spec has a Text child with same content
3. If match found:
   a. DO NOT generate Text() for that content
   b. Add accessibilityLabel to Image() instead
   c. Document in code comments: "// Text embedded in image"
```

**Cross-reference with component children:**

```
For each component being generated:
  For each Text child in spec:
    If Text content matches any imageWithTextCandidates name:
      Mark as SKIP_TEXT_GENERATION
      Add accessibility label to image instead
```

---

## Decision Table

| Condition | Action | Reason |
|-----------|--------|--------|
| Illustration has embedded text labels | Export as image, suppress text nodes | Text is baked into the image asset |
| Text is a separate overlay layer | Keep text node, export illustration separately | Text is independent of the image |
| `[contains-text]` annotation present | Skip `Text()` generation, add `accessibilityLabel` | Explicit signal from design-analyst |
| Flagged frame name matches text child | Skip `Text()` generation, add `accessibilityLabel` | Heuristic detection from code-generator |
| No flag or annotation, text is sibling | Generate both `Image()` and `Text()` normally | No duplication risk |

---

## Example Walkthrough

### Input: Implementation Spec

```markdown
### GrowthSectionView

| Property | Value |
|----------|-------|
| **Children** | TitleText, ChartIllustration |
| **Asset Children** | `IMAGE:growth-chart:6:32:354:132` |

## Flagged for LLM Review

| Node ID | Name | LLM Decision |
|---------|------|--------------|
| 6:32 | PROJECTED GROWTH | DOWNLOAD_AS_IMAGE |
```

Node 6:32 children include TEXT node "PROJECTED GROWTH" (node 6:33).

### Wrong Output (text duplicated)

```swift
// WRONG - Duplicates text that's in the image
VStack(spacing: 8) {
    Text("PROJECTED GROWTH")  // This text is already in the image!
        .font(.caption)

    Image("growth-chart")
        .resizable()
        .aspectRatio(contentMode: .fit)
}
```

### Correct Output (text suppressed)

```swift
struct GrowthSectionView: View {
    var body: some View {
        VStack(spacing: 8) {
            // TitleText SKIPPED - text "PROJECTED GROWTH" embedded in image

            // Asset: growth-chart (flagged illustration with embedded text)
            Image("growth-chart")
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(maxWidth: 354)
                .accessibilityLabel("Projected Growth chart")
        }
    }
}
```

---

## Platform-Specific Handling

### SwiftUI

```swift
// Image with embedded text
Image("growth-chart")
    .resizable()
    .aspectRatio(contentMode: .fit)
    .frame(maxWidth: 354)
    .accessibilityLabel("Projected Growth chart showing upward trend")
```

Key points:
- Use `.accessibilityLabel()` with descriptive text derived from the suppressed text content
- Add `.clipped()` to prevent overflow when needed
- Consider `.cornerRadius()` if parent has border radius
- Comment the suppression: `// Text embedded in image`

### React / Next.js

```tsx
{/* Image with embedded text - text node suppressed */}
<Image
    src="/images/growth-chart.png"
    alt="Projected Growth chart showing upward trend"
    width={354}
    height={132}
/>
```

Key points:
- Use `alt` attribute with descriptive text derived from the suppressed content
- Use `next/image` for optimized loading when in Next.js projects
- Do not render a separate `<span>` or `<p>` for the baked-in text

### HTML / CSS

```html
<!-- Image with embedded text - text node suppressed -->
<img
    src="images/growth-chart.png"
    alt="Projected Growth chart showing upward trend"
    width="354"
    height="132"
/>
```

---

## Image-with-Text Handling During Asset Children Replacement

When processing Asset Children entries in the code generator (Step 1.5), the `[contains-text]` annotation triggers special handling:

1. **Do NOT** generate a separate `Text()` view for the contained text
2. **Do** use the text content as `accessibilityLabel` for the `Image()` view

This is the explicit annotation path that complements the heuristic detection from Flagged Frames.

---

## Checklist

- [ ] Check "Flagged for LLM Review" section for `DOWNLOAD_AS_IMAGE` entries
- [ ] Check Asset Children for `[contains-text: "..."]` annotations
- [ ] Cross-reference flagged node names with Text children in component spec
- [ ] Suppress matching Text nodes from code generation
- [ ] Add `accessibilityLabel` (SwiftUI) or `alt` (React/HTML) with descriptive text
- [ ] Add code comment documenting the suppression reason
