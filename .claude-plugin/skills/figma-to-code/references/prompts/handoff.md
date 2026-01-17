# Phase 5: Handoff Prompt

This prompt is used to present the final output after validation is complete.

## Prompt Template

```markdown
## ROLE
You are a technical documentation expert. You are professionally reporting the completed Figma-to-code conversion.

## CONVERSION DATA

### Generated Code
[Final React component code]

### Validation Results
[Validation results from Phase 4]

### Planning Data
[Mapping information from Phase 2]

## CREATE HANDOFF REPORT

### 1. Summary Information

```markdown
## ✅ Conversion Complete

| Metric | Value |
|--------|-------|
| **Component** | ComponentName.tsx |
| **Accuracy** | XX.X% pixel match |
| **Iterations** | X/3 |
| **Duration** | ~X minutes |
| **Status** | Successful / With warnings / Manual required |
```

### 2. File List

```markdown
### 📁 Created Files

| File | Description | Status |
|------|-------------|--------|
| `src/components/HeroCard.tsx` | Main component | ✅ New |
| `src/components/HeroCard.test.tsx` | Unit tests | ⏳ Optional |
| `src/components/HeroCard.stories.tsx` | Storybook | ⏳ Optional |
```

### 3. Components Used

```markdown
### 🔗 Existing Component Usage

The following existing components were used in this conversion:

- `Button` (src/components/Button.tsx)
  - Props: `variant="primary"`, `size="lg"`

- `Badge` (src/components/Badge.tsx)
  - Props: `variant="success"`
```

### 4. Assumptions and Decisions

```markdown
### 📝 Assumptions Made

| # | Assumption | Reason |
|---|------------|--------|
| 1 | Font family set to 'Inter' | Font info missing in Figma |
| 2 | Hover state opacity 90% added | No hover state in design |
| 3 | Focus ring blue-500 used | Matches project standard |
```

### 5. Manual Check List

```markdown
### ⚠️ Manual Review Required

The following items could not be resolved automatically and require manual review:

- [ ] **Icon asset** — `icon-arrow.svg` not found, placeholder used
- [ ] **Color token** — `colors/accent` didn't match, `// TODO: Check color` added
- [ ] **Custom font** — 'Playfair Display' not installed, fallback used
```

### 6. Usage Example

```markdown
### 💡 Usage

\`\`\`tsx
import { HeroCard } from '@/components/HeroCard';

export default function HomePage() {
  return (
    <HeroCard
      title="Welcome"
      description="Lorem ipsum..."
      imageUrl="/hero.jpg"
      ctaText="Get Started"
      onCtaClick={() => console.log('clicked')}
    />
  );
}
\`\`\`
```

### 7. Responsive Behavior

```markdown
### 📱 Responsive Breakpoints

| Breakpoint | Behavior |
|------------|----------|
| Mobile (<640px) | Vertical layout, full width |
| Tablet (640-1024px) | Horizontal layout, 50/50 split |
| Desktop (>1024px) | Maximum 1200px, centered |
```

## OUTPUT FORMAT

Create the handoff report in Markdown format.
The user should be able to:
1. Understand what was generated
2. See what needs manual review
3. Learn how to use the component

## CRITICAL RULES

1. **Be transparent** — Don't hide assumptions and TODOs
2. **Be actionable** — Manual check list must be clear
3. **Show by example** — Always include usage example
4. **Provide metrics** — Accuracy percentage and iteration count
```

## Usage

1. After all phases are complete
2. Get validation results and code
3. Create handoff report using this prompt
4. Present to user
