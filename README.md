# Pixelbyte Agent Workflows

Claude Code plugin containing specialized agents and skills for code review, compliance checking, Figma-to-code conversion, and frontend development guidelines.

## Installation

Add to your `.claude/settings.json`:

```json
{
  "plugins": [
    "https://github.com/Rylaa/pixelbyte-agent-workflows"
  ]
}
```

## Requirements

### Figma Personal Access Token

For the figma-to-code skill, you need to set up a Figma token:

1. Go to Figma → Settings → Personal Access Tokens
2. Generate a new token
3. Set the environment variable:
   ```bash
   export FIGMA_PERSONAL_ACCESS_TOKEN="your-token-here"
   ```

## Skills

### figma-to-code

Converts Figma designs to pixel-perfect React/Tailwind code using a 5-phase workflow with 85%+ accuracy target.

**Features:**
- Figma design extraction via Pixelbyte Figma MCP
- Design token mapping (colors, typography, spacing)
- Code Connect support for component mapping
- Visual validation via Claude in Chrome MCP
- Automatic QA with iterative refinement

**Usage:**
Provide a Figma URL or mention "figma-to-code", "convert Figma", "implement design".

**5-Phase Workflow:**
1. **Context Acquisition** - Extract design structure, tokens, and screenshots
2. **Mapping & Planning** - Map Figma components to codebase components
3. **Code Generation** - Generate React/Tailwind code
4. **Visual Validation** - Compare implementation with Figma using Claude Vision
5. **Handoff** - Final documentation and TODO items

### frontend-dev-guidelines

Senior-level frontend development guidelines for React/TypeScript applications.

**Features:**
- Modern React patterns (Suspense, lazy loading, useSWR)
- TypeScript best practices
- Next.js App Router conventions
- shadcn/ui + Tailwind CSS styling
- Performance optimization techniques
- Security and error handling
- Testing strategies

**Topics Covered:**
- Component patterns with `React.FC<Props>`
- Data fetching with `useSWR` and `suspense: true`
- File organization with features directory
- Accessibility (WCAG 2.1 AA)
- Browser compatibility
- Advanced TypeScript patterns

**Usage:**
Mention "frontend guidelines", "React patterns", "TypeScript best practices" or reference when creating components, pages, or features.

## Agents

### prompt-compliance-checker

Validates that implementation matches the original prompt/request.

**Checks:**
- Does implementation match prompt requirements?
- Is existing functionality preserved?
- Are there any logical or technical errors?

**Usage:**
```
@pixelbyte-agent-workflows:prompt-compliance-checker
```

Or via Task tool:
```
Task(subagent_type="pixelbyte-agent-workflows:prompt-compliance-checker", prompt="Review my changes against the original prompt")
```

## MCP Servers

This plugin automatically configures the following MCP server:

### pixelbyte-figma-mcp

Figma API integration for design extraction and code generation.

**Tools provided:**
- `figma_get_file_structure` - Get file/node hierarchy
- `figma_get_node_details` - Get detailed node info
- `figma_generate_code` - Generate code from node
- `figma_get_design_tokens` - Extract design tokens
- `figma_get_screenshot` - Capture visual reference
- `figma_get_code_connect_map` - Get component mappings

## License

MIT
