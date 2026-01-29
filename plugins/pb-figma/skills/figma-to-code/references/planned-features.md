# Planned Features & Missing Pipeline Stages

> **Used by:** Planning reference (no agent currently loads this)

These features are identified as gaps in the current pipeline. They are documented here for future implementation planning.

---

## Missing Pipeline Stages

### 1. Animation & Transition Detection (Priority: Medium)

**Gap:** Current pipeline does not extract animation or transition data from Figma.

**Impact:** Generated code lacks hover transitions, loading animations, and micro-interactions.

**Proposed Solution:**
- Add animation detection to design-validator (Smart Animate, prototype transitions)
- Add animation section to Implementation Spec
- Code generators produce CSS transitions / SwiftUI .animation() / .transition()

### 2. Theme & Dark Mode Support (Priority: High)

**Gap:** No multi-theme or dark mode detection/generation.

**Impact:** Generated components only work in light mode.

**Proposed Solution:**
- Detect Figma "dark mode" page/frame variants
- Extract theme-aware tokens (light vs dark colors)
- Generate CSS custom properties with prefers-color-scheme
- Generate SwiftUI @Environment(\.colorScheme)

### 3. Design System Extraction (Priority: High)

**Gap:** Pipeline processes individual frames, not entire design systems.

**Impact:** Reusable tokens and component libraries are not extracted systematically.

**Proposed Solution:**
- Add design-system-extractor agent (between validator and analyst)
- Extract shared tokens, component library, variant definitions
- Generate shared theme/token files

### 4. Component Variant Completeness (Priority: Medium)

**Gap:** No validation that all Figma component variants are generated.

**Impact:** Missing hover, disabled, error, loading states.

**Proposed Solution:**
- design-analyst detects all variant properties from Figma components
- compliance-checker verifies all variants have corresponding code
- Add variant completeness % to Final Report

### 5. Storybook Story Generation (Priority: Low)

**Gap:** Storybook integration reference exists but is not connected to any agent.

**Impact:** Generated components lack interactive documentation.

**Proposed Solution:**
- code-generator-react generates .stories.tsx alongside components
- compliance-checker verifies story coverage

---

## Missing Agent Features

### 6. Code Connect MCP Integration (Priority: Medium)

**Gap:** Three Code Connect MCP tools (figma_get_code_connect_map, figma_add_code_connect_map, figma_remove_code_connect_map) are available but unused by any agent.

**Impact:** Pipeline doesn't leverage existing codebase component mappings.

**Proposed Solution:**
- design-analyst queries Code Connect for existing component mappings
- code-generator uses mapped components instead of generating from scratch
- compliance-checker validates Code Connect alignment

### 7. Vue & Kotlin Generator Implementation (Priority: Medium)

**Gap:** code-generator-vue.md and code-generator-kotlin.md are placeholders.

**Impact:** Only React and SwiftUI are supported.

**Proposed Solution:**
- Implement code-generator-vue using code-generator-react as template
- Implement code-generator-kotlin using code-generator-swiftui as template
- Create vue-patterns.md and kotlin-patterns.md references
