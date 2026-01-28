# Test: Image-with-Text Detection

## Purpose
Verify that code-generator skips duplicate text when illustration contains text.

## Input

Implementation Spec with flagged illustration:
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

## Expected Output (SwiftUI)

```swift
struct GrowthSectionView: View {
    var body: some View {
        // TitleText SKIPPED - text embedded in image

        Image("growth-chart")
            .resizable()
            .aspectRatio(contentMode: .fit)
            .frame(maxWidth: 354)
            .accessibilityLabel("Projected Growth chart")
    }
}
```

## NOT Expected (Bug)

```swift
// This is WRONG - Creates duplicate text
VStack {
    Text("PROJECTED GROWTH")  // This duplicates text in image!

    Image("growth-chart")
}
```

## Verification Steps

1. [ ] Flagged frame name matches potential Text child content
2. [ ] code-generator skips Text() for embedded text
3. [ ] Image() has accessibilityLabel instead
4. [ ] No duplicate "PROJECTED GROWTH" in output
