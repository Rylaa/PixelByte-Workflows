# Test: SwiftUI Gradient Rendering

## Input
Implementation Spec with AngularGradient:

```markdown
### Text with Gradient

**Component:** TypeSceneText
- **Gradient Type:** ANGULAR
- **Stops:**
  - 0.1673: #bc82f3 (opacity: 1.0)
  - 0.2365: #f4b9ea (opacity: 1.0)
  - 0.3518: #8d98ff (opacity: 1.0)
  - 0.5815: #aa6eee (opacity: 1.0)
  - 0.6970: #ff6777 (opacity: 1.0)
  - 0.8095: #ffba71 (opacity: 1.0)
  - 0.9241: #c686ff (opacity: 1.0)

**SwiftUI Mapping:** `AngularGradient` with 7 color stops
**Minimum iOS:** iOS 15.0+
```

## Expected SwiftUI Output

```swift
Text("Type a scene to generate your video")
    .font(.system(size: 14, weight: .regular))
    .foregroundStyle(  // NOT .foregroundColor()
        AngularGradient(
            stops: [
                Gradient.Stop(color: Color(hex: "#bc82f3"), location: 0.1673),
                Gradient.Stop(color: Color(hex: "#f4b9ea"), location: 0.2365),
                Gradient.Stop(color: Color(hex: "#8d98ff"), location: 0.3518),
                Gradient.Stop(color: Color(hex: "#aa6eee"), location: 0.5815),
                Gradient.Stop(color: Color(hex: "#ff6777"), location: 0.6970),
                Gradient.Stop(color: Color(hex: "#ffba71"), location: 0.8095),
                Gradient.Stop(color: Color(hex: "#c686ff"), location: 0.9241)
            ],
            center: .center
        )
    )
```

## Critical Requirements

1. **Use .foregroundStyle()** - NOT .foregroundColor()
2. **Preserve ALL stops** - All 7 stops must appear in output
3. **Exact locations** - 4-decimal precision (0.1673 not 0.17)
4. **Gradient.Stop format** - Must use Gradient.Stop(color:location:)

## Previous Behavior (Fixed)

**Bug that was found and fixed in code-generator-swiftui.md line 278:**

```swift
// ❌ BEFORE FIX - Lost decimal precision
Gradient.Stop(color: Color(hex: "#ff6777"), location: 0.697),

// ✅ AFTER FIX - Exact 4-decimal precision
Gradient.Stop(color: Color(hex: "#ff6777"), location: 0.6970),
```

**Issue:** The example gradient originally showed `0.697` instead of `0.6970`, losing the trailing zero. This caused pixel-level rendering differences from the Figma design.

**Impact:** Generated SwiftUI code would not match pixel-perfect Figma designs for gradients with trailing-zero locations.

**Other findings:**
- ✅ Correctly uses `.foregroundStyle()` (line 271)
- ✅ Preserves all 7 gradient stops (lines 274-280)
- ✅ Uses proper Gradient.Stop format
- ✅ Good gradient type mapping table (lines 287-294)

## Success Criteria
- ✅ code-generator-swiftui.md correctly handles gradient extraction from Implementation Spec
- ✅ Uses .foregroundStyle() for gradients (not .foregroundColor())
- ✅ Preserves ALL gradient stops
- ✅ Uses exact 4-decimal locations
- ✅ Test documents AngularGradient scenario
