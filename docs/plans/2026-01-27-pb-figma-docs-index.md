# pb-figma Documentation Index Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** pb-figma plugin'ine Claude Code best practices uygun documentation index yaklaşımı ekleyerek context window verimliliğini artırmak.

**Architecture:** Monolitik 1000 satırlık SKILL.md yerine, kısa bir core skill + lazy-load referanslar yapısına geçiş. Her agent sadece ihtiyacı olan referansları yükler.

**Tech Stack:** Markdown, Claude Code Plugin System

---

## Mevcut Durum Analizi

| Metrik | Değer |
|--------|-------|
| SKILL.md satır | 1000 |
| Toplam referans | 17 dosya |
| Toplam agent | 8 dosya |
| Toplam satır | 12,749 |

**Problem:** Her Figma dönüşümünde tüm SKILL.md yükleniyor (~1000 satır), agent'lar kendi dokümanlarını taşıyor.

---

## Task 1: docs-index.md Oluştur

**Files:**
- Create: `plugins/pb-figma/docs-index.md`

**Step 1: Index dosyasını oluştur**

```markdown
# pb-figma Documentation Index

> **Usage:** Bu dosya tüm pb-figma dokümantasyonunun haritasıdır.
> Agent'lar sadece ihtiyaç duydukları referansları @path ile yükler.

## Quick Start

- **Figma-to-Code Workflow:** @skills/figma-to-code/SKILL.md
- **Agent Pipeline:** @agents/README.md

## Agents

| Agent | Path | Purpose |
|-------|------|---------|
| design-validator | @agents/design-validator.md | Tasarım bütünlüğünü doğrula |
| design-analyst | @agents/design-analyst.md | Implementation spec oluştur |
| asset-manager | @agents/asset-manager.md | Asset'leri indir ve organize et |
| code-generator-react | @agents/code-generator-react.md | React/Tailwind kodu üret |
| code-generator-swiftui | @agents/code-generator-swiftui.md | SwiftUI kodu üret |
| code-generator-vue | @agents/code-generator-vue.md | Vue 3 kodu üret (v1.2.0) |
| code-generator-kotlin | @agents/code-generator-kotlin.md | Kotlin Compose kodu üret (v1.2.0) |
| compliance-checker | @agents/compliance-checker.md | Spec'e uyumu doğrula |
| font-manager | @agents/font-manager.md | Font'ları indir ve kur |

## References (Lazy Load)

### Core References
| Topic | Path | Used By |
|-------|------|---------|
| Token Mapping | @skills/figma-to-code/references/token-mapping.md | code-generator-* |
| Common Issues | @skills/figma-to-code/references/common-issues.md | code-generator-* |
| Error Recovery | @skills/figma-to-code/references/error-recovery.md | all agents |

### Validation References
| Topic | Path | Used By |
|-------|------|---------|
| Validation Guide | @skills/figma-to-code/references/validation-guide.md | design-validator |
| Visual Validation | @skills/figma-to-code/references/visual-validation-loop.md | compliance-checker |
| Responsive Validation | @skills/figma-to-code/references/responsive-validation.md | compliance-checker |
| Accessibility Validation | @skills/figma-to-code/references/accessibility-validation.md | compliance-checker |
| QA Report Template | @skills/figma-to-code/references/qa-report-template.md | compliance-checker |

### Development References
| Topic | Path | Used By |
|-------|------|---------|
| Code Connect Guide | @skills/figma-to-code/references/code-connect-guide.md | design-analyst |
| Figma MCP Server | @skills/figma-to-code/references/figma-mcp-server.md | all agents |
| Preview Setup | @skills/figma-to-code/references/preview-setup.md | compliance-checker |
| Test Generation | @skills/figma-to-code/references/test-generation.md | code-generator-* |
| Testing Strategy | @skills/figma-to-code/references/testing-strategy.md | code-generator-* |

### CI/CD & Integration
| Topic | Path | Used By |
|-------|------|---------|
| Storybook Integration | @skills/figma-to-code/references/storybook-integration.md | code-generator-react |
| CI/CD Integration | @skills/figma-to-code/references/ci-cd-integration.md | handoff |

## Prompt Templates

| Phase | Path | Used By |
|-------|------|---------|
| Analyze Design | @skills/figma-to-code/references/prompts/analyze-design.md | design-analyst |
| Mapping & Planning | @skills/figma-to-code/references/prompts/mapping-planning.md | design-analyst |
| Generate Component | @skills/figma-to-code/references/prompts/generate-component.md | code-generator-* |
| Validate & Refine | @skills/figma-to-code/references/prompts/validate-refine.md | compliance-checker |
| Handoff | @skills/figma-to-code/references/prompts/handoff.md | compliance-checker |

## Examples & Templates

| Type | Path |
|------|------|
| Card Component Example | @skills/figma-to-code/assets/examples/card-component.md |
| Component Template | @skills/figma-to-code/assets/templates/component.tsx.hbs |
| Stories Template | @skills/figma-to-code/assets/templates/component.stories.tsx.hbs |
```

**Step 2: Dosyayı kaydet**

Run: Dosya Write tool ile oluşturuldu.

**Step 3: Commit**

```bash
git add plugins/pb-figma/docs-index.md
git commit -m "feat(pb-figma): add documentation index for lazy loading"
```

---

## Task 2: SKILL.md Core Section Oluştur

**Files:**
- Modify: `plugins/pb-figma/skills/figma-to-code/SKILL.md`

**Step 1: SKILL.md'nin mevcut içeriğini yedekle**

```bash
cp plugins/pb-figma/skills/figma-to-code/SKILL.md plugins/pb-figma/skills/figma-to-code/SKILL.md.backup
```

**Step 2: SKILL.md'yi sadeleştir**

Mevcut 1000 satırlık dosyayı ~200 satıra indir. Şu bölümleri tut:
- Frontmatter (name, description)
- CRITICAL: Agent Invocation Required
- Pipeline Flow (görsel diagram)
- Agent invocation sequence
- Framework routing
- Report Directory
- Figma URL Parsing

Şu bölümleri kaldır (referanslara taşındı):
- Phase 1-5 detaylı açıklamalar (agent'larda mevcut)
- MCP Tool detayları (figma-mcp-server.md'de mevcut)
- Output Format (code-generator-base.md'de mevcut)
- WCAG 2.1 AA Accessibility (accessibility-validation.md'de mevcut)
- Common Issues (common-issues.md'de mevcut)
- DOM Flattening Rules (code-generator-base.md'ye taşı)
- TODO Comment Strategy (code-generator-base.md'ye taşı)

**Step 3: Index referansı ekle**

SKILL.md'nin başına ekle:

```markdown
## Documentation

For detailed references, see @docs-index.md

Key references loaded on-demand:
- Token conversion: @references/token-mapping.md
- Common issues: @references/common-issues.md
- Error recovery: @references/error-recovery.md
```

**Step 4: Commit**

```bash
git add plugins/pb-figma/skills/figma-to-code/SKILL.md
git commit -m "refactor(pb-figma): slim down SKILL.md with lazy-load references"
```

---

## Task 3: code-generator-base.md Güncelle

**Files:**
- Modify: `plugins/pb-figma/agents/code-generator-base.md`

**Step 1: DOM Flattening ve TODO Strategy ekle**

SKILL.md'den taşınan bölümleri ekle:

```markdown
## DOM Flattening Rules

Skip unnecessary layers:
```
❌ SKIP:
- Frames used only for grouping (no bg/border/padding)
- Wrapper containers with single child
- Groups with no visual effect
- Default-named empty wrappers ("Frame 1", "Group 2")

✅ CONVERT TO DIV:
- Has background color
- Has border or shadow
- Has padding/margin
- Has border-radius
```

## TODO Comment Strategy

For missing or ambiguous values:
```tsx
// TODO: Check color - Design token 'colors/accent' not in theme
const accentColor = "text-blue-500"; // Temporary value

// TODO: Check font - 'Custom Font' not installed
const fontFamily = "font-sans"; // Fallback

// TODO: Check icon - Asset not found
<PlaceholderIcon className="w-6 h-6" />
```
```

**Step 2: Reference loading talimatı ekle**

```markdown
## Reference Loading

Before generating code, load these references as needed:

- **Token conversion issues?** → Read @references/token-mapping.md
- **Layout problems?** → Read @references/common-issues.md
- **Error during generation?** → Read @references/error-recovery.md
```

**Step 3: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-base.md
git commit -m "feat(pb-figma): add DOM flattening and TODO strategy from SKILL.md"
```

---

## Task 4: Agent'lara Lazy Loading Talimatı Ekle

**Files:**
- Modify: `plugins/pb-figma/agents/design-validator.md`
- Modify: `plugins/pb-figma/agents/design-analyst.md`
- Modify: `plugins/pb-figma/agents/compliance-checker.md`
- Modify: `plugins/pb-figma/agents/code-generator-react.md`
- Modify: `plugins/pb-figma/agents/code-generator-swiftui.md`

**Step 1: Her agent'a reference loading section ekle**

Her agent dosyasının başına (frontmatter'dan sonra, ilk heading'den önce) ekle:

```markdown
## Reference Loading

Load these references when needed:
- Error recovery: @skills/figma-to-code/references/error-recovery.md
- [Agent-specific references from docs-index.md]
```

**design-validator.md için:**
```markdown
## Reference Loading

Load these references when needed:
- Validation guide: @skills/figma-to-code/references/validation-guide.md
- Error recovery: @skills/figma-to-code/references/error-recovery.md
```

**design-analyst.md için:**
```markdown
## Reference Loading

Load these references when needed:
- Code Connect guide: @skills/figma-to-code/references/code-connect-guide.md
- Mapping planning prompt: @skills/figma-to-code/references/prompts/mapping-planning.md
- Error recovery: @skills/figma-to-code/references/error-recovery.md
```

**compliance-checker.md için:**
```markdown
## Reference Loading

Load these references when needed:
- Visual validation loop: @skills/figma-to-code/references/visual-validation-loop.md
- QA report template: @skills/figma-to-code/references/qa-report-template.md
- Responsive validation: @skills/figma-to-code/references/responsive-validation.md
- Accessibility validation: @skills/figma-to-code/references/accessibility-validation.md
- Error recovery: @skills/figma-to-code/references/error-recovery.md
```

**code-generator-react.md için:**
```markdown
## Reference Loading

Load these references when needed:
- Token mapping: @skills/figma-to-code/references/token-mapping.md
- Common issues: @skills/figma-to-code/references/common-issues.md
- Test generation: @skills/figma-to-code/references/test-generation.md
- Storybook integration: @skills/figma-to-code/references/storybook-integration.md
- Error recovery: @skills/figma-to-code/references/error-recovery.md
```

**code-generator-swiftui.md için:**
```markdown
## Reference Loading

Load these references when needed:
- Token mapping: @skills/figma-to-code/references/token-mapping.md
- Common issues: @skills/figma-to-code/references/common-issues.md
- Test generation: @skills/figma-to-code/references/test-generation.md
- Error recovery: @skills/figma-to-code/references/error-recovery.md
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/*.md
git commit -m "feat(pb-figma): add lazy loading reference sections to agents"
```

---

## Task 5: SKILL.md Slim Version Yaz

**Files:**
- Modify: `plugins/pb-figma/skills/figma-to-code/SKILL.md`

**Step 1: Yeni SKILL.md içeriği**

```markdown
---
name: figma-to-code
description: This skill handles pixel-perfect Figma design conversion to production code (React/Tailwind, SwiftUI, Vue, Kotlin) using Pixelbyte Figma MCP Server. It should be used when a Figma URL or design selection needs to be converted to production-ready code. The skill employs a 5-phase workflow with framework detection and routing to framework-specific agents.
---

# Figma-to-Code Pixel-Perfect Conversion

## Documentation Index

For detailed references, see @docs-index.md

## ⚠️ CRITICAL: Agent Invocation Required

**DO NOT use MCP tools directly.** This skill orchestrates specialized agents via `Task` tool.

```
┌─────────────────────────────────────────────────────────────────┐
│  YOU MUST USE Task TOOL TO INVOKE AGENTS                        │
│                                                                  │
│  ❌ WRONG: Calling mcp__pixelbyte-figma-mcp__* directly         │
│  ✅ RIGHT: Task(subagent_type="pb-figma:design-validator", ...) │
└─────────────────────────────────────────────────────────────────┘
```

## Agent Pipeline

```
Figma URL
    │
    ▼
┌─────────────────────────┐
│ 1. design-validator     │ → Validation Report
└─────────────────────────┘
    │
    ▼
┌─────────────────────────┐
│ 2. design-analyst       │ → Implementation Spec
└─────────────────────────┘
    │
    ▼
┌─────────────────────────┐
│ 3. asset-manager        │ → Updated Spec + Assets
└─────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│ 4. Framework Detection & Routing            │
├─────────────────────────────────────────────┤
│ → React/Next.js  → code-generator-react     │
│ → SwiftUI        → code-generator-swiftui   │
│ → Vue/Nuxt       → code-generator-vue       │
│ → Kotlin/Compose → code-generator-kotlin    │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────┐
│ 5. compliance-checker   │ → Final Report
└─────────────────────────┘
```

## Invocation Sequence

```python
# Step 1: Design Validator
Task(subagent_type="pb-figma:design-validator",
     prompt="Validate Figma URL: {url}")

# Step 2: Design Analyst (MANDATORY - creates Implementation Spec)
Task(subagent_type="pb-figma:design-analyst",
     prompt="Create Implementation Spec from: docs/figma-reports/{file_key}-validation.md")

# Step 3: Asset Manager
Task(subagent_type="pb-figma:asset-manager",
     prompt="Download assets from spec: docs/figma-reports/{file_key}-spec.md")

# Step 4: Code Generator (framework-specific)
Task(subagent_type="pb-figma:code-generator-{framework}",
     prompt="Generate code from spec: docs/figma-reports/{file_key}-spec.md")

# Step 5: Compliance Checker
Task(subagent_type="pb-figma:compliance-checker",
     prompt="Validate implementation against spec: docs/figma-reports/{file_key}-spec.md")
```

## Framework Detection

Before dispatching code generator (Step 4):

1. **Swift/Xcode:** `ls *.xcodeproj *.xcworkspace Package.swift` → `code-generator-swiftui`
2. **Android/Kotlin:** Find `build.gradle.kts` with `androidx.compose` → `code-generator-kotlin`
3. **Node.js:** Check `package.json` for react/next/vue/nuxt → route accordingly
4. **Default:** `code-generator-react` with warning

## Report Directory

All reports saved to: `docs/figma-reports/`

```
docs/figma-reports/
├── {file_key}-validation.md   # Agent 1 output
├── {file_key}-spec.md         # Agent 2+3 output
└── {file_key}-final.md        # Agent 5 output
```

## Figma URL Parsing

```
URL: figma.com/design/ABC123xyz/MyDesign?node-id=456-789

file_key: ABC123xyz
node_id: 456:789 (convert hyphen "-" to colon ":")

⚠️ URL format "456-789" must be used as "456:789"!
```

## Prerequisites

- Pixelbyte Figma MCP with `FIGMA_PERSONAL_ACCESS_TOKEN`
- Claude in Chrome MCP for visual validation
- Node.js runtime

## Core Principles

1. **Never guess** — Always extract design tokens from MCP
2. **Use semantic HTML** — Prefer correct elements over `<div>` soup
3. **Apply Claude Vision validation** — Visual comparison with TodoWrite tracking
4. **Match exactly** — No creative interpretation, match the design precisely
5. **Leverage Code Connect** — Map Figma components to existing codebase components

## References

For detailed information on specific topics:

| Topic | Reference |
|-------|-----------|
| Token conversion | @references/token-mapping.md |
| Common issues | @references/common-issues.md |
| Visual validation | @references/visual-validation-loop.md |
| Error recovery | @references/error-recovery.md |
| Figma MCP tools | @references/figma-mcp-server.md |
| Code Connect | @references/code-connect-guide.md |
```

**Step 2: Satır sayısını doğrula**

```bash
wc -l plugins/pb-figma/skills/figma-to-code/SKILL.md
# Hedef: ~150-200 satır
```

**Step 3: Commit**

```bash
git add plugins/pb-figma/skills/figma-to-code/SKILL.md
git commit -m "refactor(pb-figma): reduce SKILL.md from 1000 to ~150 lines with lazy refs"
```

---

## Task 6: Backup Dosyasını Sil ve Test Et

**Files:**
- Delete: `plugins/pb-figma/skills/figma-to-code/SKILL.md.backup`

**Step 1: Yeni yapıyı test et**

```bash
# Dosya yapısını kontrol et
ls -la plugins/pb-figma/docs-index.md
ls -la plugins/pb-figma/skills/figma-to-code/SKILL.md

# Satır sayılarını karşılaştır
echo "=== Before ==="
wc -l plugins/pb-figma/skills/figma-to-code/SKILL.md.backup
echo "=== After ==="
wc -l plugins/pb-figma/skills/figma-to-code/SKILL.md
```

**Step 2: Backup'ı sil**

```bash
rm plugins/pb-figma/skills/figma-to-code/SKILL.md.backup
```

**Step 3: Final commit**

```bash
git add -A
git commit -m "chore(pb-figma): cleanup backup file after docs-index migration"
```

---

## Task 7: CHANGELOG Güncelle

**Files:**
- Modify: `plugins/pb-figma/CHANGELOG.md`

**Step 1: Yeni versiyon entry ekle**

```markdown
## [1.5.0] - 2026-01-27

### Added
- Documentation index (`docs-index.md`) for lazy-loading references
- Reference loading sections in all agents

### Changed
- Reduced SKILL.md from 1000 lines to ~150 lines
- Moved DOM flattening rules to code-generator-base.md
- Moved TODO comment strategy to code-generator-base.md

### Performance
- Estimated 60-70% reduction in initial context loading
- Agents now load references on-demand instead of upfront
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/CHANGELOG.md
git commit -m "docs(pb-figma): update CHANGELOG for v1.5.0 docs-index feature"
```

---

## Summary

| Task | Files | Satır Değişimi |
|------|-------|----------------|
| 1 | docs-index.md | +100 (yeni) |
| 2-5 | SKILL.md | -850 (1000→150) |
| 3 | code-generator-base.md | +50 |
| 4 | 5 agent dosyası | +50 (toplam) |
| 7 | CHANGELOG.md | +15 |

**Net sonuç:** ~635 satır azalma (skill yüklenirken)

**Token tasarrufu:** Her session'da ~5000-7000 token tasarruf (tahmini)
