# PB Workflows

Claude Code plugin collection with modular installation. Install only what you need.

## Installation

Add the marketplace to your `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": [
    "https://github.com/Rylaa/pixelbyte-agent-workflows"
  ]
}
```

Then install individual plugins:

```bash
# Install all plugins
claude plugin install pb-figma
claude plugin install pb-frontend
claude plugin install pb-code-review

# Or install only what you need
claude plugin install pb-figma  # Just Figma-to-code
```

## Available Plugins

| Plugin | Description | Category |
|--------|-------------|----------|
| `pb-figma` | Figma-to-code conversion with pixel-perfect accuracy | Design |
| `pb-frontend` | Senior-level frontend development guidelines | Development |
| `pb-code-review` | Prompt compliance checker agent | Quality |

---

## pb-figma

Converts Figma designs to pixel-perfect React/Tailwind code using a 5-phase workflow with 85%+ accuracy target.

### Requirements

Set up a Figma Personal Access Token:

```bash
export FIGMA_PERSONAL_ACCESS_TOKEN="your-token-here"
```

Get your token: Figma → Settings → Personal Access Tokens

### Features

- Figma design extraction via Pixelbyte Figma MCP
- Design token mapping (colors, typography, spacing)
- Code Connect support for component mapping
- Visual validation via Claude in Chrome MCP
- Automatic QA with iterative refinement

### 5-Phase Workflow

1. **Context Acquisition** - Extract design structure, tokens, and screenshots
2. **Mapping & Planning** - Map Figma components to codebase components
3. **Code Generation** - Generate React/Tailwind code
4. **Visual Validation** - Compare implementation with Figma using Claude Vision
5. **Handoff** - Final documentation and TODO items

### Usage

Provide a Figma URL or mention "figma-to-code", "convert Figma", "implement design".

### MCP Server

Automatically configures `pixelbyte-figma-mcp` with these tools:
- `figma_get_file_structure` - Get file/node hierarchy
- `figma_get_node_details` - Get detailed node info
- `figma_generate_code` - Generate code from node
- `figma_get_design_tokens` - Extract design tokens
- `figma_get_screenshot` - Capture visual reference
- `figma_get_code_connect_map` - Get component mappings

---

## pb-frontend

Senior-level frontend development guidelines for React/TypeScript applications.

### Features

- Modern React patterns (Suspense, lazy loading, useSWR)
- TypeScript best practices
- Next.js App Router conventions
- shadcn/ui + Tailwind CSS styling
- Performance optimization techniques
- Security and error handling
- Testing strategies

### Topics Covered

- Component patterns with `React.FC<Props>`
- Data fetching with `useSWR` and `suspense: true`
- File organization with features directory
- Accessibility (WCAG 2.1 AA)
- Browser compatibility (Safari focus)
- Advanced TypeScript patterns

### Usage

Automatically invoked when creating components, pages, or features. Mention "frontend guidelines", "React patterns", or "TypeScript best practices".

---

## pb-code-review

Validates that implementation matches the original prompt/request.

### Checks

- Does implementation match prompt requirements?
- Is existing functionality preserved (regression detection)?
- Are there any logical or technical errors?
- Evidence-based feedback with file paths and line numbers

### Usage

Via Task tool:
```
Task(subagent_type="pb-code-review:prompt-compliance-checker", prompt="Review my changes against the original prompt")
```

Or ask: "Check if my changes match what was requested"

---

## License

MIT
