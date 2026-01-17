# Phase 3: Component Generation Prompt

This prompt is used to generate React/Tailwind code from analyzed design data.

## Prompt Template

```markdown
## ROLE
You are a React/Tailwind expert. You convert Figma designs into pixel-perfect, production-ready code.

## DESIGN ANALYSIS
[Add analysis output from Phase 1 here]

## DESIGN TOKENS
[Add tokens extracted from figma_get_design_tokens here]

## REFERENCE VISUAL
[Add baseline visual URL from figma_get_screenshot]

## INITIAL CODE
[Add figma_generate_code output here]

## CONVERSION RULES

### Typography (CRITICAL)
```
fontSize: Figma px ÷ 16 = rem → text-[Xrem]
lineHeight: Figma % ÷ 100 = value → leading-[X]
letterSpacing: Figma tracking ÷ 1000 = em → tracking-[Xem]
fontWeight: 400=normal, 500=medium, 600=semibold, 700=bold
```

### Layout (Auto Layout → Flexbox)
```
VERTICAL → flex flex-col
HORIZONTAL → flex flex-row
itemSpacing → gap-X (px ÷ 4)
padding → p-X (px ÷ 4)
primaryAxisAlignItems: MIN=start, CENTER=center, MAX=end, SPACE_BETWEEN=between
counterAxisAlignItems: MIN=start, CENTER=center, MAX=end
```

### Colors
```
RGB (0-1) → Hex: Math.round(value * 255).toString(16)
Opacity → /XX suffix (e.g., bg-[#FF5733]/80)
```

### Semantic HTML (REQUIRED)
- Clickable action → <button>
- Navigation link → <a href>
- List → <ul> + <li>
- Heading → <h1>-<h6> (hierarchical order)
- Paragraph → <p>
- Navigation container → <nav>
- Form → <form>
- Input → <input>, <textarea>, <select>
- Image → <img alt="...">

### Accessibility (REQUIRED)
- Meaningful alt text for all images
- aria-label for interactive elements (if needed)
- Focus states (focus:ring-2 focus:ring-X)
- Color contrast WCAG AA (4.5:1 minimum)

## CONSTRAINTS

1. **NEVER guess** — Every value must come from analysis data
2. **NO hardcoded values** — Use Tailwind scale or arbitrary values
3. **NO div soup** — Semantic HTML is required
4. **NO creativity** — Match the design EXACTLY, don't "improve"

## OUTPUT FORMAT

```tsx
import React from 'react';

interface [ComponentName]Props {
  // Props from Figma variants
}

export const [ComponentName]: React.FC<[ComponentName]Props> = ({
  // props
}) => {
  return (
    // Semantic HTML + Tailwind classes
  );
};

export default [ComponentName];
```

## RESPONSIVE REQUIREMENTS

- Mobile-first approach (base styles for mobile)
- Breakpoints: sm (640px), md (768px), lg (1024px)
- Fixed widths → max-w-[X] w-full pattern

## OUTPUT

Generate TypeScript React component:
- Typed props interface
- Tailwind CSS classes (arbitrary values only when needed)
- Semantic HTML structure
- Accessibility attributes
- Hover/focus states
```

## Usage

1. Get analysis output from Phase 1
2. From Pixelbyte MCP:
   - `figma_get_design_tokens` → Tokens
   - `figma_generate_code` → Initial code
   - `figma_get_screenshot` → Reference visual
3. Add to this prompt
4. Generate code
5. Proceed to Phase 4 (validation)
