# Phase 2: Mapping & Planning Prompt

This prompt is used for planning BEFORE writing code after analyzing the design data.

## Prompt Template

```markdown
## ROLE
You are an architecture planning expert. You examine Figma design data and determine the optimal strategy for code generation.

## DESIGN ANALYSIS
[Add analysis output from Phase 1 here]

## PROJECT ENVIRONMENT

### Existing Components
[List src/components/ contents]

### Tailwind Configuration
[Theme info from tailwind.config.js]

### Libraries Used
[Relevant dependencies from package.json]

## PLANNING TASKS

### 1. Existing Component Matching

Check for existing components for each Figma element:

```
Figma Node: "Primary Button"
├── Existing component? → src/components/Button.tsx ✓
├── Props match? → variant="primary" ✓
└── Decision: USE (no recreation)

Figma Node: "Hero Card"
├── Existing component? → ✗
├── Similar component? → src/components/Card.tsx (partial)
└── Decision: CREATE NEW (extend Card)
```

### 2. Token Mapping Table

```json
{
  "colors": {
    "VariableID:123": {
      "figmaName": "colors/primary",
      "tailwindClass": "bg-primary",
      "fallback": "bg-blue-600",
      "status": "matched"
    },
    "VariableID:456": {
      "figmaName": "colors/accent",
      "tailwindClass": null,
      "fallback": "bg-amber-500",
      "status": "TODO"
    }
  },
  "spacing": {
    "16px": "4",
    "24px": "6",
    "32px": "8"
  },
  "typography": {
    "Heading/H1": "text-4xl font-bold",
    "Body/Regular": "text-base font-normal"
  }
}
```

### 3. Layout Strategy

```
Root Container:
├── Layout: flex-col (VERTICAL)
├── Responsive: md:flex-row (>768px horizontal)
├── Gap: gap-6 (24px)
└── Padding: p-8 (32px)

Child Elements:
├── Image: w-full md:w-1/2 (responsive width)
├── Content: flex-1 (fill remaining)
└── Button: w-full md:w-auto (responsive)
```

### 4. Responsive Planning

```
Mobile First Approach:

Base (320px+):
- flex-col
- w-full
- text-center
- p-4

sm (640px+):
- p-6

md (768px+):
- flex-row
- text-left
- p-8

lg (1024px+):
- max-w-6xl mx-auto
```

### 5. Accessibility Plan

```
Semantic Tags:
├── "Card Container" → <article>
├── "Title" → <h2>
├── "Description" → <p>
├── "CTA Button" → <button> or <a>
└── "Image" → <img alt="...">

ARIA Requirements:
├── Icon buttons → aria-label
├── Decorative images → aria-hidden="true"
└── Form elements → label connection
```

## OUTPUT FORMAT

```json
{
  "componentPlan": {
    "name": "HeroCard",
    "type": "new",
    "extendsFrom": null,
    "dependencies": ["Button", "Badge"]
  },
  "reusedComponents": [
    {
      "figmaNode": "Primary Button",
      "componentPath": "src/components/Button.tsx",
      "props": {
        "variant": "primary",
        "size": "lg"
      }
    }
  ],
  "newComponents": [
    {
      "name": "HeroCard",
      "semanticTag": "article",
      "layout": "flex-col md:flex-row"
    }
  ],
  "tokenMappings": {
    "matched": 12,
    "fallback": 3,
    "todo": 1
  },
  "responsiveBreakpoints": ["base", "md", "lg"],
  "accessibilityNotes": [
    "Image alt text required",
    "Add button focus ring"
  ]
}
```

## OUTPUT

Provide planning output in JSON format.
This plan will be followed in Phase 3 for code generation.

## CRITICAL RULES

1. **Never recreate existing components** — Import and use
2. **Mark TODO if token not found** — Use fallback, add comment
3. **Make responsive plan mobile-first** — Base styles for mobile
4. **Plan semantic HTML** — div is last resort
```

## Usage

1. Get analysis output from Phase 1
2. Examine project environment (components, config)
3. Add to this prompt
4. Get planning output
5. Proceed to Phase 3 (code generation)
