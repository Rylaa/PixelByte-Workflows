# Phase 1: Design Analysis Prompt

This prompt is used to analyze Figma design data.

## Prompt Template

```markdown
## ROLE
You are acting as a Figma design analyst. You will analyze the given design data and extract the necessary information for code generation.

## DESIGN DATA
[Add figma_get_node_details and figma_generate_code responses here]

## DESIGN TOKENS
[Add figma_get_design_tokens response here]

## ANALYSIS TASKS

### 1. Component Hierarchy
List all elements in the design:
- Root container properties
- Children elements and their types (FRAME, TEXT, RECTANGLE, etc.)
- Nesting depth

### 2. Layout Analysis
Extract Auto Layout information:
- layoutMode: VERTICAL or HORIZONTAL
- itemSpacing (gap value)
- padding values (top, bottom, left, right)
- primaryAxisAlignItems (justify-content)
- counterAxisAlignItems (align-items)

### 3. Design Tokens

**Colors:**
- For each fill: RGB values → Hex conversion
- Opacity values
- If gradient exists: direction and stops

**Typography:**
- fontFamily
- fontSize (px)
- fontWeight
- lineHeightPercent (%)
- letterSpacing

**Effects:**
- Shadow: offset, radius, spread, color
- Border: width, color, radius

### 4. Semantic Analysis
Determine appropriate HTML tag for each element:
- Clickable appearance → button or a
- Text list → ul/li
- Heading appearance → h1-h6
- Image → img
- Input field → input/textarea

### 5. Responsive Hints
- Fixed widths vs. flexible widths
- Minimum/maximum sizes
- Breakpoint suggestions

## OUTPUT FORMAT

```json
{
  "componentName": "...",
  "rootLayout": {
    "direction": "flex-col | flex-row",
    "gap": "gap-X",
    "padding": "p-X | px-X py-X",
    "justify": "justify-X",
    "align": "items-X"
  },
  "tokens": {
    "colors": {
      "background": "#XXXXXX",
      "text": "#XXXXXX",
      ...
    },
    "typography": {
      "heading": {
        "size": "text-X",
        "weight": "font-X",
        "lineHeight": "leading-X",
        "tracking": "tracking-X"
      },
      ...
    },
    "spacing": {
      "gap": "X",
      "padding": "X"
    },
    "effects": {
      "shadow": "shadow-X | shadow-[...]",
      "borderRadius": "rounded-X"
    }
  },
  "elements": [
    {
      "name": "...",
      "figmaType": "TEXT | FRAME | ...",
      "htmlTag": "h1 | p | button | ...",
      "content": "...",
      "styles": "..."
    }
  ],
  "responsive": {
    "fixedWidth": true | false,
    "suggestedBreakpoints": ["sm", "md", "lg"]
  }
}
```
```

## Usage

1. Call Pixelbyte MCP tools:
   - `figma_get_file_structure` → Get file structure
   - `figma_get_node_details` → Get node details
   - `figma_get_design_tokens` → Extract tokens
   - `figma_generate_code` → Get initial code
2. Add responses to this prompt
3. Get analysis output
4. Proceed to Phase 2 (mapping & planning)
