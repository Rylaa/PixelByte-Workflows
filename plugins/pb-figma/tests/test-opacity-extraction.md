# Test: Opacity Extraction

## Purpose
Verify Design Analyst correctly extracts BOTH fills[0].opacity AND node.opacity from Figma API, then calculates effective opacity.

## Input
Figma node with compound opacity:
- **fills[0].opacity**: 0.4 (fill layer opacity)
- **node.opacity**: 1.0 (node-level opacity)
- **Expected effective opacity**: 0.4 × 1.0 = 0.4

## API Response Example
```json
{
  "fills": [
    {
      "type": "SOLID",
      "color": { "r": 1.0, "g": 1.0, "b": 1.0 },
      "opacity": 0.4
    }
  ],
  "opacity": 1.0
}
```

## Expected Output in Implementation Spec

Design Tokens table should show:

| Property | Color | Opacity | Usage |
|----------|-------|---------|-------|
| Border | #ffffff | 0.4 | `.stroke(Color.white.opacity(0.4))` |

## Calculation Formula
```
effectiveOpacity = fills[0].opacity × node.opacity
effectiveOpacity = 0.4 × 1.0 = 0.4
```

## Edge Cases to Handle

### Case 1: Both opacities present
- fills[0].opacity = 0.5
- node.opacity = 0.8
- Expected: 0.5 × 0.8 = 0.4

### Case 2: Fill opacity missing (defaults to 1.0)
- fills[0].opacity = undefined
- node.opacity = 0.6
- Expected: 1.0 × 0.6 = 0.6

### Case 3: Both default to 1.0
- fills[0].opacity = undefined
- node.opacity = undefined
- Expected: 1.0 × 1.0 = 1.0

## Bug Detection Criteria
Test FAILS if:
- Only fills[0].opacity is extracted (missing node.opacity multiplication)
- Only node.opacity is extracted (missing fills[0].opacity)
- Opacity column shows wrong value
- Opacity defaults incorrectly when values missing

## Real-World Test Case
**Node:** bt65gbJ6sSdKRP4x3IY151, 10203:16369 (TypeSceneText)

Expected opacities:
- Background #150200: opacity 1.0
- Radial gradient overlay: opacity 0.2
- Border stroke white: opacity 0.4
- Text fill gradient: opacity 1.0
