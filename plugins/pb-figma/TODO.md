# pb-figma Plugin - TODO / Improvements

## High Priority

### 1. figma_list_assets Tool Enhancement

**Problem:** Tool cannot distinguish between icons with same name, leading to incorrect asset identification.

**Current Behavior:**
```
| Frame 2121316303 | 3:230 | SVG |  ← All have same name
| Frame 2121316303 | 3:299 | SVG |
| Frame 2121316303 | 3:291 | SVG |
```

**Proposed Enhancement:**

```python
# Add checkmark pattern detection
CHECKMARK_PATTERNS = [
    "M19.7 24.5L22.9 27.7L29.3 21.3",  # Common checkmark path
    "L.*L.*",  # Generic check pattern (two line segments)
]

def classify_icon_content(svg_content: str) -> str:
    """Classify icon by SVG path content."""
    for pattern in CHECKMARK_PATTERNS:
        if re.search(pattern, svg_content):
            return "STATUS_INDICATOR"
    return "THEMATIC"

# Add parent position analysis
def get_icon_position(node_id: str, parent_children: list) -> str:
    """Determine if icon is leading or trailing in parent layout."""
    index = next(i for i, c in enumerate(parent_children) if c.id == node_id)
    if index == 0:
        return "leading"
    elif index == len(parent_children) - 1:
        return "trailing"
    return "middle"
```

**Expected Output:**
```
| Asset | Node ID | Position | Icon Type |
|-------|---------|----------|-----------|
| Frame 2121316303 | 3:230 | trailing | STATUS_INDICATOR |
| Frame 2121316303 | 3:299 | trailing | STATUS_INDICATOR |
| Frame 2121316303 | 3:291 | trailing | STATUS_INDICATOR |
| icon-clock | 3:XXX | leading | THEMATIC |
| icon-flask | 3:YYY | leading | THEMATIC |
```

---

## Medium Priority

### 2. Asset Deduplication

When multiple assets have identical names, auto-generate unique suffixes:
- `Frame-001.svg`, `Frame-002.svg`
- Or use parent context: `card1-icon.svg`, `card2-icon.svg`

### 3. SVG Content Preview

Add optional SVG path preview in asset inventory to help identify icon content without downloading.

---

## Completed

- [x] Border opacity extraction (v1.4.0)
- [x] Gradient text support (v1.4.0)
- [x] Text decoration support (v1.4.0)
- [x] Design Analyst card icon classification rules (v1.4.1)
- [x] Design Validator duplicate icon handling (v1.4.1)
