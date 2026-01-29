# Pixelbyte Figma MCP Server Reference

> **Used by:** all agents

Pixelbyte Figma MCP Server is an MCP server that integrates with the Figma API.

## Server Configuration

**MCP Configuration:**

```json
{
  "mcpServers": {
    "pixelbyte-figma-mcp": {
      "command": "uvx",
      "args": ["pixelbyte-figma-mcp"],
      "env": {
        "FIGMA_PERSONAL_ACCESS_TOKEN": "your-figma-token"
      }
    }
  }
}
```

## Setup

### Getting Figma Personal Access Token

1. Figma → Settings → Personal Access Tokens
2. Click "Generate new token"
3. Give the token a descriptive name
4. Copy the token and save it securely
5. Set as `FIGMA_PERSONAL_ACCESS_TOKEN` environment variable

---

## Tools Reference

### figma_get_file_structure

Gets the structure and node hierarchy of a Figma file.

**Parameters:**
```typescript
{
  params: {
    file_key: string,      // Figma file key (from URL)
    depth?: number,        // 1-10, default: 2
    response_format?: "markdown" | "json"
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_get_file_structure({
  params: {
    file_key: "ABC123xyz",
    depth: 3,
    response_format: "markdown"
  }
})
```

**Returns:**
- Page list
- Frame and component names
- Node IDs
- Hierarchical structure

---

### figma_get_node_details

Gets comprehensive details about a specific node including all design properties.

**Parameters:**
```typescript
{
  params: {
    file_key: string,      // Figma file key
    node_id: string,       // Node ID (e.g., "1:2")
    response_format?: "markdown" | "json"
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_get_node_details({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456",
    response_format: "json"
  }
})
```

**Returns:**
- Dimensions and position
- Fills (solid, gradient, image)
- Strokes (color, weight, align, cap, join, dashes)
- Effects (shadows, blurs)
- Auto-layout properties
- Corner radii (individual corners)
- Constraints (responsive behavior)
- Transform (rotation, scale)
- Component/Instance info
- Bound variables
- Blend mode

---

### figma_generate_code

Generates detailed code from a Figma node with all nested children.

**Parameters:**
```typescript
{
  params: {
    file_key: string,      // Figma file key
    node_id: string,       // Node ID
    framework?: "react" | "react_tailwind" | "vue" | "vue_tailwind" | "html_css" | "tailwind_only" | "css" | "scss" | "swiftui" | "kotlin",
    component_name?: string  // Custom component name (auto-generated from node name if not provided)
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_generate_code({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456",
    framework: "react_tailwind",
    component_name: "HeroCard"
  }
})
```

**Returns:**
- Production-ready code for the specified framework
- All nested children
- Text content
- Styles (colors, shadows, borders)
- Layout information

**Framework Options:**

| Framework | Description |
|-----------|-------------|
| react | React component (vanilla CSS) |
| react_tailwind | React + Tailwind CSS |
| vue | Vue 3 component (Composition API) |
| vue_tailwind | Vue 3 + Tailwind CSS |
| html_css | Standard HTML + CSS |
| tailwind_only | Just Tailwind CSS classes |
| css | Pure CSS with all styles |
| scss | SCSS with variables and nesting |
| swiftui | iOS SwiftUI Views |
| kotlin | Android Jetpack Compose |

---

### figma_get_design_tokens

Extracts design tokens (colors, typography, spacing, effects) from a Figma file.

**Parameters:**
```typescript
{
  params: {
    file_key: string,
    node_id?: string,              // Optional - specific node to analyze
    include_colors?: boolean,       // default: true
    include_typography?: boolean,   // default: true
    include_spacing?: boolean,      // default: true
    include_effects?: boolean,      // default: true
    include_generated_code?: boolean // default: true - Include ready-to-use CSS variables, SCSS variables, and Tailwind config
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_get_design_tokens({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456",
    include_colors: true,
    include_typography: true,
    include_spacing: true,
    include_effects: true,
    include_generated_code: true
  }
})
```

**Returns:**
```json
{
  "colors": [
    { "name": "primary", "hex": "#FE4601", "rgba": "rgba(254,70,1,1)" }
  ],
  "typography": [
    { "fontFamily": "Inter", "fontSize": 16, "fontWeight": 500, "lineHeight": 1.5 }
  ],
  "spacing": [
    { "name": "padding", "value": 16 }
  ],
  "effects": [
    { "type": "DROP_SHADOW", "color": "rgba(0,0,0,0.1)", "offset": { "x": 0, "y": 4 } }
  ],
  "generatedCode": {
    "cssVariables": "...",
    "scssVariables": "...",
    "tailwindConfig": "..."
  }
}
```

---

### figma_get_styles

Retrieves all published styles from a Figma file.

**Parameters:**
```typescript
{
  params: {
    file_key: string,
    include_fill_styles?: boolean,    // default: true - Include fill/color styles
    include_text_styles?: boolean,    // default: true - Include text/typography styles
    include_effect_styles?: boolean,  // default: true - Include effect styles (shadows, blurs)
    include_grid_styles?: boolean,    // default: true - Include grid/layout styles
    response_format?: "markdown" | "json"
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_get_styles({
  params: {
    file_key: "ABC123xyz",
    include_fill_styles: true,
    include_text_styles: true,
    include_effect_styles: true,
    include_grid_styles: true,
    response_format: "markdown"
  }
})
```

**Returns:**
- Published color styles
- Published text styles
- Published effect styles
- Published grid styles

**Use Case:** Fetch reusable design tokens defined in Figma that are published as styles.

---

### figma_get_screenshot

Takes screenshots of specific nodes from a Figma file.

**Parameters:**
```typescript
{
  params: {
    file_key: string,
    node_ids: string[],          // Array: ["1:2", "3:4"] (max 10)
    format?: "png" | "svg" | "jpg" | "pdf",  // default: "png"
    scale?: number               // 0.01 - 4.0, default: 2
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_get_screenshot({
  params: {
    file_key: "ABC123xyz",
    node_ids: ["123:456"],
    format: "png",
    scale: 2
  }
})
```

**Returns:**
- Local file paths for each screenshot
- URLs valid for 30 days

**⚠️ Important:** Node ID format must be `123:456` (colon, not hyphen).

---

### figma_list_assets

Lists all exportable assets in a Figma file or node with smart icon detection.

**Parameters:**
```typescript
{
  params: {
    file_key: string,
    node_id?: string,              // Optional - search within specific node
    include_images?: boolean,       // default: true - Include image fills
    include_icons?: boolean,        // default: true - Smart detect icon frames (recommended)
    include_vectors?: boolean,      // default: false - Include raw vector paths (usually not needed)
    include_exports?: boolean,      // default: true - Include nodes with export settings
    response_format?: "markdown" | "json"
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_list_assets({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456",
    include_images: true,
    include_icons: true,
    include_vectors: false,
    include_exports: true,
    response_format: "markdown"
  }
})
```

**Returns:**
- Image fills (photos, illustrations)
- Icon frames (smart detected by name pattern like 'mynaui:icon' or by size/structure)
- Raw vector nodes (if enabled)
- Nodes with export settings configured

**Smart Icon Detection:** Uses intelligent heuristics to detect icon frames and treats them as single assets instead of drilling into individual vector paths.

---

### figma_get_images

Gets actual downloadable URLs for image fills in a Figma file.

**Parameters:**
```typescript
{
  params: {
    file_key: string,
    node_id?: string    // Optional - specific node to get images from
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_get_images({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456"
  }
})
```

**Returns:**
- Real S3 URLs that can be downloaded
- Image references mapped to URLs
- URLs valid for 30 days

**Use Case:** Resolves internal imageRef values to real downloadable URLs.

---

### figma_export_assets

Batch exports assets from Figma nodes.

**Parameters:**
```typescript
{
  params: {
    file_key: string,
    node_ids: string[],              // Array of node IDs to export (max 50)
    format?: "png" | "svg" | "jpg" | "pdf",  // default: "png"
    scale?: number,                   // 0.01 - 4.0, default: 2
    include_svg_for_vectors?: boolean // default: true - Generate inline SVG for vector nodes
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_export_assets({
  params: {
    file_key: "ABC123xyz",
    node_ids: ["123:456", "789:012"],
    format: "svg",
    scale: 2,
    include_svg_for_vectors: true
  }
})
```

**Returns:**
- Export URLs for each node
- Generated inline SVGs for vector nodes (if enabled)

**Use Case:** Batch export icons, images, or other assets from Figma.

---

### figma_get_code_connect_map

Gets Code Connect mappings for a Figma file.

**Parameters:**
```typescript
{
  params: {
    file_key: string,
    node_id?: string    // Optional - specific node to get mapping for
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_get_code_connect_map({
  params: {
    file_key: "ABC123xyz"
  }
})
```

**Returns:**
```json
{
  "mappings": [
    {
      "node_id": "123:456",
      "component_path": "src/components/Button.tsx",
      "component_name": "Button",
      "props_mapping": { "Variant": "variant" },
      "example": "<Button variant=\"primary\">Click</Button>"
    }
  ]
}
```

**Use Case:** Retrieve stored mappings between Figma components and code implementations for accurate code generation.

---

### figma_add_code_connect_map

Adds or updates a Code Connect mapping for a Figma component.

**Parameters:**
```typescript
{
  params: {
    file_key: string,
    node_id: string,
    component_path: string,       // e.g., "src/components/Button.tsx"
    component_name: string,       // e.g., "Button"
    props_mapping?: object,       // { "Variant": "variant", "Size": "size" }
    variants?: object,            // { "primary": { "variant": "primary" } }
    example?: string              // Usage code snippet
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_add_code_connect_map({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456",
    component_path: "src/components/Button.tsx",
    component_name: "Button",
    props_mapping: { "Variant": "variant", "Size": "size" },
    variants: { "primary": { "variant": "primary" } },
    example: "<Button variant='primary'>Click</Button>"
  }
})
```

**Use Case:** Create mappings between Figma components and their code implementations for better code generation.

---

### figma_remove_code_connect_map

Removes a Code Connect mapping for a Figma component.

**Parameters:**
```typescript
{
  params: {
    file_key: string,
    node_id: string
  }
}
```

**Usage:**
```javascript
mcp__pixelbyte-figma-mcp__figma_remove_code_connect_map({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456"
  }
})
```

---

## Tools Summary

| Tool | Purpose |
|------|---------|
| `figma_get_file_structure` | Get file hierarchy and node tree |
| `figma_get_node_details` | Get comprehensive node properties |
| `figma_generate_code` | Generate production-ready code |
| `figma_get_design_tokens` | Extract colors, typography, spacing, effects |
| `figma_get_styles` | Get published styles from file |
| `figma_get_screenshot` | Take node screenshots |
| `figma_list_assets` | List exportable assets with smart icon detection |
| `figma_get_images` | Get downloadable image URLs |
| `figma_export_assets` | Batch export assets |
| `figma_get_code_connect_map` | Get component-to-code mappings |
| `figma_add_code_connect_map` | Add component-to-code mapping |
| `figma_remove_code_connect_map` | Remove component-to-code mapping |

---

## Figma URL Parsing

### URL Format

```
https://www.figma.com/design/xHgE5Ab7cD9fG1hI/ProjectName?node-id=123-456&t=abc

file_key: xHgE5Ab7cD9fG1hI  (part between design/ and / or ?)
node_id: 123:456            (hyphen → colon)
```

### Parsing Examples

| URL Part | Value |
|----------|-------|
| `design/ABC123/...` | file_key: `ABC123` |
| `?node-id=1-2` | node_id: `1:2` |
| `?node-id=123-456` | node_id: `123:456` |

⚠️ **Critical Error:** The hyphen (`-`) in `node-id=123-456` from the URL must be converted to a colon (`:`) in API calls: `123:456`

---

## Workflow Recommendation

### Phase 1: Gathering Information

```
1. Extract file_key and node_id from URL
2. figma_get_file_structure → Understand structure
3. figma_get_node_details → Get detailed properties
4. figma_get_design_tokens → Extract tokens
5. figma_get_styles → Get published styles
6. figma_generate_code → Initial code
7. figma_get_code_connect_map → Existing mappings
8. figma_get_screenshot → Visual reference
```

### Asset Workflow

```
1. figma_list_assets → Catalog all assets in design
2. figma_get_images → Get image fill URLs
3. figma_export_assets → Batch export icons/images
```

### Phase 4: Validation

```
1. figma_get_screenshot → Get Figma visual
2. Take browser screenshot with Claude in Chrome MCP → Get implementation visual
3. Compare with Claude Vision
4. List differences with TodoWrite
```

---

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Invalid file key" | Make sure correct file_key is extracted from URL |
| "Node not found" | Check node_id format: hyphen → colon |
| "Rate limit" | Wait between requests, use cache |
| "Token invalid" | Check FIGMA_PERSONAL_ACCESS_TOKEN |
| "Permission denied" | Check access permission to Figma file |

---

## Best Practices

1. **Use cache** - Don't make repeated requests for the same file
2. **Batch operations** - Take screenshots for multiple nodes at once
3. **Depth limit** - Depth 2-3 is sufficient for `figma_get_file_structure`
4. **Scale 2** - Scale: 2 is recommended for screenshots (Retina quality)
5. **Node ID format** - Always use `:`, never use `-`
6. **Smart icon detection** - Use `figma_list_assets` with `include_icons: true` for better icon handling
7. **Published styles** - Use `figma_get_styles` to get design system tokens
