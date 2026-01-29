# pb-figma Comprehensive System Improvements Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** pb-figma plugin sistemindeki tüm kalite, tutarlılık ve eksiklikleri gidermek — agent prompt bloat azaltma, cross-reference düzeltmeleri, handoff format standardizasyonu, dil tutarlılığı, SKILL.md path düzeltmeleri, checkpoint sistemi, yapılandırılabilirlik ve eksik pipeline aşamalarını kapsar.

**Architecture:** Bu plan 7 faz halinde organize edilmiştir. Her faz bağımsız commit'ler üretir. Dosyalar Markdown formatındadır — agent prompt dosyaları (`agents/*.md`), referans dosyalar (`skills/figma-to-code/references/*.md`), ve orkestrasyon dosyaları (`skills/figma-to-code/SKILL.md`, `docs-index.md`). Değişiklikler geriye uyumludur.

**Tech Stack:** Markdown, YAML frontmatter, Glob-based reference loading, Figma MCP tools

**Base Path:** `/Users/yusufdemirkoparan/Projects/pixelbyte-agent-workflows/plugins/pb-figma`

---

## FAZ 1: Path Tutarsızlıkları & Referans Düzeltmeleri (Quick Wins)

### Task 1: SKILL.md @references Path'lerini Glob-Based Sisteme Güncelle

**Files:**
- Modify: `skills/figma-to-code/SKILL.md:133-145`

**Context:** SKILL.md hâlâ `@references/token-mapping.md` ve `@docs-index.md` gibi eski path formatını kullanıyor. Tüm agent dosyaları zaten `Glob("**/references/{filename}.md")` formatına geçirildi. SKILL.md'yi aynı standarda getirmeliyiz.

**Step 1: Mevcut References tablosunu oku**

`SKILL.md` dosyasının 133-145 satırlarını oku. Şu anda şöyle görünüyor:

```markdown
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

**Step 2: @references path'lerini Glob formatına çevir**

Yukarıdaki tabloyu şu şekilde güncelle:

```markdown
## References

**How to load references:** Use `Glob("**/references/{filename}.md")` to find the absolute path, then `Read()` the result.

| Topic | Reference File | Glob Pattern |
|-------|---------------|--------------|
| Token conversion | `token-mapping.md` | `**/references/token-mapping.md` |
| Common issues | `common-issues.md` | `**/references/common-issues.md` |
| Visual validation | `visual-validation-loop.md` | `**/references/visual-validation-loop.md` |
| Error recovery | `error-recovery.md` | `**/references/error-recovery.md` |
| Figma MCP tools | `figma-mcp-server.md` | `**/references/figma-mcp-server.md` |
| Code Connect | `code-connect-guide.md` | `**/references/code-connect-guide.md` |
```

**Step 3: `@docs-index.md` referansını güncelle**

`SKILL.md` satır 10'daki `@docs-index.md` referansını da düzelt:

```markdown
## Documentation Index

For detailed references, load via Glob: `**/docs-index.md`
```

**Step 4: Değişikliği doğrula**

Run: `grep -n "@references\|@docs-index\|@skills" plugins/pb-figma/skills/figma-to-code/SKILL.md`
Expected: Hiçbir `@references/`, `@docs-index`, veya `@skills/` pattern'i kalmamalı.

**Step 5: Commit**

```bash
git add plugins/pb-figma/skills/figma-to-code/SKILL.md
git commit -m "fix(skill): replace @references paths with Glob-based loading in SKILL.md"
```

---

### Task 2: asset-manager.md'ye Reference Loading Bölümü Ekle

**Files:**
- Modify: `agents/asset-manager.md:14-16`

**Context:** Diğer tüm agent'lar (design-validator, design-analyst, code-generator-react, code-generator-swiftui, compliance-checker) YAML frontmatter'dan sonra bir "Reference Loading" bölümü içeriyor. asset-manager.md bu bölüme sahip değil. Tutarlılık için eklenmeli.

**Step 1: asset-manager.md'nin mevcut YAML frontmatter sonrasını oku**

Satır 14-20 civarını oku. Şu anda YAML kapanışından (`---`) hemen sonra `# Asset Manager Agent` başlığı geliyor.

**Step 2: Reference Loading bölümünü ekle**

YAML frontmatter kapanışı (`---` satır 14) ile `# Asset Manager Agent` başlığı arasına şunu ekle:

```markdown

## Reference Loading

**How to load references:** Use `Glob("**/references/{filename}.md")` to find the absolute path, then `Read()` the result. Do NOT use `@skills/...` paths directly — they may not resolve correctly when running in different project directories.

Load these references when needed:
- Asset classification: `asset-classification-guide.md` → Glob: `**/references/asset-classification-guide.md`
- Illustration detection: `illustration-detection.md` → Glob: `**/references/illustration-detection.md`
- Error recovery: `error-recovery.md` → Glob: `**/references/error-recovery.md`

```

**Step 3: Dosya içindeki eski inline referansları kontrol et**

Run: `grep -n "asset-classification-guide\|illustration-detection\|error-recovery" plugins/pb-figma/agents/asset-manager.md`
Expected: Mevcut inline referanslar da bulunacak (bunlar dokunulmayacak, sadece üstteki Reference Loading bölümü eklendi).

**Step 4: Commit**

```bash
git add plugins/pb-figma/agents/asset-manager.md
git commit -m "fix(asset-manager): add Reference Loading section for consistency"
```

---

### Task 3: font-manager.md'ye Reference Loading Bölümü Ekle

**Files:**
- Modify: `agents/font-manager.md:19-21`

**Context:** font-manager.md de Reference Loading bölümüne sahip değil. Sadece `font-handling.md` referansı kullanılıyor ama format tutarlılığı için standart bölüm eklenmeli.

**Step 1: font-manager.md'nin YAML frontmatter sonrasını oku**

Satır 19-25 civarını oku.

**Step 2: Reference Loading bölümünü ekle**

YAML frontmatter kapanışı (`---`) ile `# Font Manager Agent` başlığı arasına şunu ekle:

```markdown

## Reference Loading

**How to load references:** Use `Glob("**/references/{filename}.md")` to find the absolute path, then `Read()` the result. Do NOT use `@skills/...` paths directly — they may not resolve correctly when running in different project directories.

Load these references when needed:
- Font handling: `font-handling.md` → Glob: `**/references/font-handling.md`
- Error recovery: `error-recovery.md` → Glob: `**/references/error-recovery.md`

```

**Step 3: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "fix(font-manager): add Reference Loading section for consistency"
```

---

### Task 4: design-analyst.md Ölü Referansı Temizle

**Files:**
- Modify: `agents/design-analyst.md:20`

**Context:** design-analyst.md satır 20'de `mapping-planning.md` referansı var ancak docs-index.md bu dosyayı "⚠️ Unused" olarak işaretliyor. Prompt template'leri artık kullanılmıyor.

**Step 1: Ölü referansı oku**

Satır 20: `- Mapping planning prompt: `mapping-planning.md` → Glob: `**/references/prompts/mapping-planning.md``

**Step 2: Ölü referansı kaldır**

Satır 20'yi tamamen sil. Reference Loading bölümü şu hale gelmeli:

```markdown
Load these references when needed:
- Code Connect guide: `code-connect-guide.md` → Glob: `**/references/code-connect-guide.md`
- Error recovery: `error-recovery.md` → Glob: `**/references/error-recovery.md`
```

**Step 3: Dosyada başka mapping-planning referansı olmadığını doğrula**

Run: `grep -n "mapping-planning" plugins/pb-figma/agents/design-analyst.md`
Expected: Hiç sonuç çıkmamalı.

**Step 4: Commit**

```bash
git add plugins/pb-figma/agents/design-analyst.md
git commit -m "fix(design-analyst): remove dead mapping-planning.md reference"
```

---

## FAZ 2: Dil Tutarlılığı (TR/EN Standardizasyonu)

### Task 5: docs-index.md Dil Standardizasyonu

**Files:**
- Modify: `docs-index.md` (97 satır)

**Context:** docs-index.md Türkçe ve İngilizce karışık kullanıyor. Agent "Purpose" sütunu Türkçe, başlıklar ve notlar karışık. Tüm belgeyi İngilizce'ye standardize edelim çünkü tüm referans dosyaları ve agent prompt'ları İngilizce.

**Step 1: docs-index.md'yi tamamen oku**

Tüm 97 satırı oku.

**Step 2: Türkçe içerikleri İngilizce'ye çevir**

Aşağıdaki değişiklikleri yap:

1. Satır 3: `> **Usage:** Bu dosya tüm pb-figma dokümantasyonunun haritasıdır.` → `> **Usage:** This file is the documentation map for the entire pb-figma plugin. Agents load only the references they need via @path.`

2. Agent tablosu Purpose sütunu:
   - `Tasarım bütünlüğünü doğrula` → `Validate design completeness`
   - `Implementation spec oluştur` → `Create Implementation Spec`
   - `Asset'leri indir ve organize et` → `Download and organize assets`
   - `React/Tailwind kodu üret` → `Generate React/Tailwind code`
   - `SwiftUI kodu üret` → `Generate SwiftUI code`
   - `Vue 3 kodu üret` → `Generate Vue 3 code`
   - `Kotlin Compose kodu üret` → `Generate Kotlin Compose code`
   - `Spec'e uyumu doğrula` → `Validate spec compliance`
   - `Font'ları indir ve kur` → `Download and configure fonts`

3. Satır 25: `> **Note:** Vue ve Kotlin generator'ları gelecek sürümler için planlanmıştır. Şu an için React veya SwiftUI generator'larını kullanın.` → `> **Note:** Vue and Kotlin generators are planned for future releases. Use React or SwiftUI generators for now.`

4. Satır 79: `> **Note:** Bu prompt template'leri önceki versiyonlar için tasarlandı, ancak şu an hiçbir agent tarafından kullanılmıyor. Referans için korunuyor.` → `> **Note:** These prompt templates were designed for previous versions and are not currently used by any agent. Kept for reference.`

5. Satır 89: `**Aktif agent'lar referansları doğrudan yükler** - aşağıdaki "References" bölümüne bakın.` → `**Active agents load references directly** — see the "References" section above.`

**Step 3: Doğrula**

Run: `grep -niP "[çşğüöıİ]" plugins/pb-figma/docs-index.md`
Expected: Hiçbir Türkçe karakter kalmamalı.

**Step 4: Commit**

```bash
git add plugins/pb-figma/docs-index.md
git commit -m "fix(docs-index): standardize language to English for consistency"
```

---

### Task 6: agents/README.md Dil Kontrolü

**Files:**
- Modify: `agents/README.md` (61 satır)

**Step 1: agents/README.md'yi tamamen oku**

Türkçe içerik var mı kontrol et.

**Step 2: Türkçe varsa İngilizce'ye çevir**

Varsa tüm Türkçe ifadeleri İngilizce'ye çevir. Yoksa bu task'ı atla.

**Step 3: Doğrula**

Run: `grep -niP "[çşğüöıİ]" plugins/pb-figma/agents/README.md`
Expected: Hiçbir Türkçe karakter kalmamalı.

**Step 4: Commit (değişiklik varsa)**

```bash
git add plugins/pb-figma/agents/README.md
git commit -m "fix(agents): standardize README language to English"
```

---

## FAZ 3: Agent Prompt Bloat Azaltma

### Task 7: code-generator-react.md Frame Properties İçeriğini Referansa Taşı

**Files:**
- Modify: `agents/code-generator-react.md` (~satır 191-384, ~193 satır)
- Verify: `skills/figma-to-code/references/frame-properties.md` (350 satır, zaten mevcut)

**Context:** code-generator-react.md içinde "Frame Properties Map" bölümü (~193 satır) inline olarak yazılmış. Bu bilgi zaten `frame-properties.md` referans dosyasında mevcut. Agent'taki inline kopyayı kaldırıp referansa yönlendirme eklemeliyiz.

**Step 1: code-generator-react.md'deki Frame Properties bölümünü oku**

Satır 191'den başlayarak Frame Properties Map bölümünü tamamen oku. Bölümün başlangıç ve bitiş satırlarını not et.

**Step 2: frame-properties.md referans dosyasını oku**

Referans dosyasının React/Tailwind örneklerini içerip içermediğini doğrula. İçeriyorsa, agent'taki inline kopya güvenle kaldırılabilir.

**Step 3: Inline içeriği referans yönlendirmesiyle değiştir**

Agent'taki Frame Properties Map bölümünü (başlangıç heading dahil) şununla değiştir:

```markdown
## Frame Properties Mapping

> **Reference:** Load `frame-properties.md` via `Glob("**/references/frame-properties.md")` for dimension extraction, corner radius (uniform/per-corner), border/stroke handling, and Tailwind mapping rules.

### Quick Reference (React/Tailwind)

| Figma Property | Tailwind/CSS |
|---------------|-------------|
| cornerRadius (uniform) | `rounded-{size}` |
| cornerRadius (per-corner) | `rounded-tl-{} rounded-tr-{} rounded-br-{} rounded-bl-{}` |
| strokeWeight + strokeAlign INSIDE | `border-{width}` + `box-border` |
| strokeWeight + strokeAlign OUTSIDE | `outline outline-{width}` or wrapper `div` |
| strokeWeight + strokeAlign CENTER | `border-{width}` (visual approximation) |
```

**Step 4: Satır sayısı değişimini doğrula**

Run: `wc -l plugins/pb-figma/agents/code-generator-react.md`
Expected: Satır sayısı ~170+ azalmış olmalı (1341 → ~1170 civarı).

**Step 5: Reference Loading bölümüne frame-properties ekle**

Satır 15-24'teki Reference Loading listesine ekle (eğer yoksa):
```markdown
- Frame properties: `frame-properties.md` → Glob: `**/references/frame-properties.md`
```

**Step 6: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-react.md
git commit -m "refactor(code-gen-react): extract Frame Properties inline content to reference"
```

---

### Task 8: code-generator-react.md Text Decoration İçeriğini Referansa Taşı

**Files:**
- Modify: `agents/code-generator-react.md` (~satır 587-657, ~70 satır)
- Verify: `skills/figma-to-code/references/text-decoration.md` (319 satır, zaten mevcut)

**Context:** Text decoration bölümü inline yazılmış. `text-decoration.md` referansı zaten kapsamlı.

**Step 1: Text decoration bölümünü oku**

code-generator-react.md'deki text decoration bölümünü oku.

**Step 2: text-decoration.md'nin React örnekleri içerdiğini doğrula**

Referans dosyasında Tailwind/CSS text decoration mapping'leri olduğunu doğrula.

**Step 3: Inline içeriği referans yönlendirmesiyle değiştir**

```markdown
## Text Decoration

> **Reference:** Load `text-decoration.md` via `Glob("**/references/text-decoration.md")` for underline/strikethrough mapping, inline text variations, and character style override handling.

### Quick Reference (React/Tailwind)

| Figma textDecoration | Tailwind Class |
|---------------------|---------------|
| UNDERLINE | `underline` |
| STRIKETHROUGH | `line-through` |
| UNDERLINE + STRIKETHROUGH | `underline line-through` |
```

**Step 4: Reference Loading bölümüne text-decoration ekle (eğer yoksa)**

```markdown
- Text decoration: `text-decoration.md` → Glob: `**/references/text-decoration.md`
```

**Step 5: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-react.md
git commit -m "refactor(code-gen-react): extract Text Decoration inline content to reference"
```

---

### Task 9: code-generator-react.md Gradient Handling İçeriğini Referansa Taşı

**Files:**
- Modify: `agents/code-generator-react.md` (~satır 574-585 ve ilgili detaylar)
- Verify: `skills/figma-to-code/references/gradient-handling.md` (387 satır)

**Step 1: Gradient bölümünü oku ve referansla karşılaştır**

**Step 2: Inline içeriği referans yönlendirmesiyle değiştir**

```markdown
## Gradient Handling

> **Reference:** Load `gradient-handling.md` via `Glob("**/references/gradient-handling.md")` for gradient type detection, stop position precision (4 decimal), and CSS/Tailwind gradient syntax.

### Quick Reference (React/Tailwind)

| Gradient Type | CSS Pattern |
|--------------|-------------|
| LINEAR | `bg-gradient-to-{dir}` or `background: linear-gradient({angle}deg, ...)` |
| RADIAL | `background: radial-gradient(...)` |
| Multiple stops | Use `from-{color} via-{color} to-{color}` for 3-stop, CSS for complex |
```

**Step 3: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-react.md
git commit -m "refactor(code-gen-react): extract Gradient Handling inline content to reference"
```

---

### Task 10: code-generator-react.md CVA Patterns ve Required Utilities Bölümlerini Referansa Taşı

**Files:**
- Modify: `agents/code-generator-react.md` (~satır 889-1299, ~410 satır)
- Create: `skills/figma-to-code/references/react-patterns.md`

**Context:** CVA (class-variance-authority) patterns (~87 satır) ve Required Utilities (cn() utility, CSS variables, Tailwind 4 theme setup — ~107 satır) React'e özgü domain knowledge içeriyor. Bu bilgi yeni bir `react-patterns.md` referans dosyasına taşınmalı.

**Step 1: code-generator-react.md'deki CVA ve Utilities bölümlerini oku**

Satır 889-976 (CVA) ve 1192-1299 (Required Utilities) aralıklarını tamamen oku.

**Step 2: Yeni referans dosyası oluştur**

`skills/figma-to-code/references/react-patterns.md` dosyasını oluştur:

```markdown
# React/Tailwind Patterns Reference

> **Used by:** code-generator-react, compliance-checker

## CVA (class-variance-authority) Patterns

[CVA içeriğini buraya taşı — tam kopyası]

## Required Utilities

### cn() Utility

[cn() utility içeriğini buraya taşı — tam kopyası]

### CSS Variables Setup

[CSS variables içeriğini buraya taşı — tam kopyası]

### Tailwind 4 Theme Setup

[Tailwind 4 theme setup içeriğini buraya taşı — tam kopyası]
```

**Step 3: Agent'taki inline içeriği referans yönlendirmesiyle değiştir**

CVA bölümü:
```markdown
## Component Variants (CVA)

> **Reference:** Load `react-patterns.md` via `Glob("**/references/react-patterns.md")` for CVA pattern implementation, variant definition, and compound variants.
```

Required Utilities bölümü:
```markdown
## Required Utilities

> **Reference:** Load `react-patterns.md` via `Glob("**/references/react-patterns.md")` for cn() utility, CSS variables setup, and Tailwind 4 theme configuration.
```

**Step 4: Reference Loading bölümüne ekle**

```markdown
- React patterns: `react-patterns.md` → Glob: `**/references/react-patterns.md`
```

**Step 5: docs-index.md'ye yeni referansı ekle**

Development References tablosuna ekle:

```markdown
| React Patterns | @skills/figma-to-code/references/react-patterns.md | code-generator-react |
```

**Step 6: Satır sayısı değişimini doğrula**

Run: `wc -l plugins/pb-figma/agents/code-generator-react.md`
Expected: ~400 satır azalma (toplamda ~1341 → ~740 civarı Faz 3 sonunda).

**Step 7: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-react.md plugins/pb-figma/skills/figma-to-code/references/react-patterns.md plugins/pb-figma/docs-index.md
git commit -m "refactor(code-gen-react): extract CVA patterns and utilities to react-patterns.md reference"
```

---

### Task 11: code-generator-swiftui.md Frame Properties İçeriğini Referansa Taşı

**Files:**
- Modify: `agents/code-generator-swiftui.md` (~satır 323-675, ~352 satır)
- Verify: `skills/figma-to-code/references/frame-properties.md`

**Context:** SwiftUI agent'taki Frame Properties bölümü React'inkinden 83% daha uzun (352 vs 193 satır). frame-properties.md referansına SwiftUI bölümü eklenip inline kaldırılmalı.

**Step 1: SwiftUI frame properties bölümünü oku**

Satır 323-675 aralığını tamamen oku. SwiftUI-spesifik pattern'leri not et (.frame(), UnevenRoundedRectangle, stroke alignment modifiers).

**Step 2: frame-properties.md'ye SwiftUI bölümü olup olmadığını kontrol et**

Referans dosyasını oku. SwiftUI örnekleri yoksa veya eksikse, bu bilgiyi referansa ekle.

**Step 3: frame-properties.md'ye SwiftUI bölümünü ekle (gerekiyorsa)**

Referans dosyasının sonuna SwiftUI-spesifik pattern'leri ekle:

```markdown
## SwiftUI Implementation

### Frame Modifier Patterns

[SwiftUI frame, cornerRadius, stroke pattern'lerini ekle]

### Per-Corner Radius (iOS 16+)

[UnevenRoundedRectangle pattern'ini ekle]

### Stroke Alignment Modifiers

[INSIDE/OUTSIDE/CENTER stroke SwiftUI pattern'lerini ekle]
```

**Step 4: Agent'taki inline içeriği referans yönlendirmesiyle değiştir**

```markdown
## Frame Properties Mapping

> **Reference:** Load `frame-properties.md` via `Glob("**/references/frame-properties.md")` for dimension extraction, corner radius (uniform/per-corner with UnevenRoundedRectangle for iOS 16+), border/stroke handling (INSIDE/OUTSIDE/CENTER alignment), and SwiftUI .frame() modifier patterns.

### Quick Reference (SwiftUI)

| Figma Property | SwiftUI Modifier |
|---------------|-----------------|
| width/height | `.frame(width:height:)` |
| cornerRadius (uniform) | `.clipShape(RoundedRectangle(cornerRadius:))` |
| cornerRadius (per-corner) | `.clipShape(UnevenRoundedRectangle(...))` (iOS 16+) |
| strokeWeight INSIDE | `.overlay(RoundedRectangle(...).stroke(...))` |
| strokeWeight OUTSIDE | `ZStack { RoundedRectangle().stroke(...); content }` |
```

**Step 5: Satır sayısı değişimini doğrula**

Run: `wc -l plugins/pb-figma/agents/code-generator-swiftui.md`
Expected: ~320+ satır azalma.

**Step 6: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-swiftui.md plugins/pb-figma/skills/figma-to-code/references/frame-properties.md
git commit -m "refactor(code-gen-swiftui): extract Frame Properties inline content to reference"
```

---

### Task 12: code-generator-swiftui.md Glass Effect, Layer Order, Adaptive Layout Bölümlerini Referansa Taşı

**Files:**
- Modify: `agents/code-generator-swiftui.md` (~satır 728-843 Glass+Layer ~115 satır, ~satır 1281-1384 Adaptive ~103 satır)
- Modify: `skills/figma-to-code/references/shadow-blur-effects.md` (Glass Effect bilgisi)
- Modify: `skills/figma-to-code/references/layer-order-hierarchy.md` (Layer Order bilgisi)
- Create: `skills/figma-to-code/references/swiftui-patterns.md` (Adaptive Layout + SwiftUI-spesifik)

**Context:** Glass Effect bilgisi `shadow-blur-effects.md`'de zaten var. Layer Order bilgisi `layer-order-hierarchy.md`'de var. Adaptive Layout SwiftUI'ye özgü ve yeni bir referans dosyasına taşınmalı.

**Step 1: Üç bölümü oku**

1. Glass Effect (satır 728-793)
2. Layer Order (satır 794-843)
3. Adaptive Layout (satır 1281-1384)

**Step 2: shadow-blur-effects.md ve layer-order-hierarchy.md'nin SwiftUI içeriği olduğunu doğrula**

Her iki referans dosyasını oku. SwiftUI-spesifik örnekler eksikse ekle.

**Step 3: swiftui-patterns.md referans dosyası oluştur**

```markdown
# SwiftUI Patterns Reference

> **Used by:** code-generator-swiftui, compliance-checker

## Adaptive Layout Patterns

### iPad/Tablet Support

[Adaptive layout içeriğini taşı — content width capping, LazyVGrid, maxWidth patterns]

## Text Sizing & Auto-Resize

[Text sizing pattern'lerini taşı — HEIGHT/TRUNCATE/NONE/WIDTH_AND_HEIGHT mapping]

## Required Extensions

### Color+Hex Extension

[Color extension kodunu taşı]

### RoundedCorner Shape (iOS 15)

[RoundedCorner shape kodunu taşı]
```

**Step 4: Agent'taki inline içerikleri referans yönlendirmeleriyle değiştir**

Glass Effect:
```markdown
## Glass Effect Implementation

> **Reference:** Load `shadow-blur-effects.md` via `Glob("**/references/shadow-blur-effects.md")` for glass effect detection heuristics, iOS 26 Liquid Glass, and .ultraThinMaterial fallback patterns.
```

Layer Order:
```markdown
## Layer Order & ZStack

> **Reference:** Load `layer-order-hierarchy.md` via `Glob("**/references/layer-order-hierarchy.md")` for children array ordering rules, ZStack positioning, and zIndex assignment.
```

Adaptive Layout:
```markdown
## Adaptive Layout (iPad/Tablet)

> **Reference:** Load `swiftui-patterns.md` via `Glob("**/references/swiftui-patterns.md")` for content width capping, LazyVGrid card lists, and flexible width patterns.
```

**Step 5: docs-index.md'ye yeni referansı ekle**

```markdown
| SwiftUI Patterns | @skills/figma-to-code/references/swiftui-patterns.md | code-generator-swiftui |
```

**Step 6: Reference Loading bölümüne ekle**

```markdown
- SwiftUI patterns: `swiftui-patterns.md` → Glob: `**/references/swiftui-patterns.md`
```

**Step 7: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-swiftui.md plugins/pb-figma/skills/figma-to-code/references/shadow-blur-effects.md plugins/pb-figma/skills/figma-to-code/references/layer-order-hierarchy.md plugins/pb-figma/skills/figma-to-code/references/swiftui-patterns.md plugins/pb-figma/docs-index.md
git commit -m "refactor(code-gen-swiftui): extract Glass, Layer Order, Adaptive Layout to references"
```

---

### Task 13: code-generator-swiftui.md Text Sizing, Inline Variations, Required Extensions Bölümlerini Referansa Taşı

**Files:**
- Modify: `agents/code-generator-swiftui.md` (~satır 1105-1279 Inline+Text ~174 satır, ~satır 1698-1766 Extensions ~68 satır)
- Modify: `skills/figma-to-code/references/swiftui-patterns.md` (Task 12'de oluşturuldu)

**Step 1: İlgili bölümleri oku**

1. Inline Text Variations (satır 1105-1197)
2. Text Sizing & Auto-Resize (satır 1210-1279)
3. Required Extensions (satır 1698-1766)

**Step 2: swiftui-patterns.md'ye ekle**

Task 12'de oluşturulan `swiftui-patterns.md` dosyasına bu bölümleri ekle.

**Step 3: Agent'taki inline içerikleri referans yönlendirmeleriyle değiştir**

Inline Text:
```markdown
## Inline Text Variations

> **Reference:** Load `text-decoration.md` via `Glob("**/references/text-decoration.md")` for inline text variation handling and character style overrides.
> Also load `swiftui-patterns.md` via `Glob("**/references/swiftui-patterns.md")` for SwiftUI Text concatenation with `+` operator.
```

Text Sizing:
```markdown
## Text Sizing & Auto-Resize

> **Reference:** Load `swiftui-patterns.md` via `Glob("**/references/swiftui-patterns.md")` for textAutoResize mode mapping to .lineLimit() and .truncationMode().
```

Required Extensions:
```markdown
## Required Extensions

> **Reference:** Load `swiftui-patterns.md` via `Glob("**/references/swiftui-patterns.md")` for Color+Hex extension and RoundedCorner shape implementations.
```

**Step 4: Final satır sayısını doğrula**

Run: `wc -l plugins/pb-figma/agents/code-generator-swiftui.md`
Expected: Tüm Faz 3 sonunda ~1952 → ~1100 civarı (850+ satır azalma).

**Step 5: Commit**

```bash
git add plugins/pb-figma/agents/code-generator-swiftui.md plugins/pb-figma/skills/figma-to-code/references/swiftui-patterns.md
git commit -m "refactor(code-gen-swiftui): extract Text, Extensions to swiftui-patterns.md reference"
```

---

## FAZ 4: Handoff Format Standardizasyonu & Flagged Frames Netleştirme

### Task 14: Pipeline Handoff Format Dokümanı Oluştur

**Files:**
- Create: `skills/figma-to-code/references/pipeline-handoff.md`
- Modify: `docs-index.md`

**Context:** Agent'lar arası handoff formatları tutarsız. design-validator çıktı formatı, design-analyst'in beklediği formatla tam örtüşmüyor. Standart bir handoff formatı dokümanı oluşturmalıyız.

**Step 1: Mevcut agent input/output formatlarını incele**

Her agent'ın Input ve Output bölümlerini oku:
- design-validator.md: Output formatı (Validation Report)
- design-analyst.md: Input beklentisi (Validation Report) ve Output formatı (Implementation Spec)
- asset-manager.md: Input beklentisi (Implementation Spec) ve Output formatı (Updated Spec)
- code-generator-*.md: Input beklentisi (Updated Spec)
- compliance-checker.md: Input beklentisi (Spec + Generated Code)

**Step 2: pipeline-handoff.md oluştur**

```markdown
# Pipeline Handoff Formats Reference

> **Used by:** All pipeline agents

## Overview

This document defines the exact data contract between pipeline stages. Each agent must produce output matching the next agent's expected input.

## Stage 1 → Stage 2: Validation Report

**Producer:** design-validator
**Consumer:** design-analyst

**File:** `docs/figma-reports/{file_key}-validation.md`

### Required Sections

| Section | Required | Description |
|---------|----------|-------------|
| File Info | ✅ | file_key, node_id, URL |
| Status | ✅ | PASS / WARN / FAIL |
| Design Tokens | ✅ | Colors, typography, spacing tables |
| Frame Properties | ✅ | Dimensions, corner radius, borders |
| Assets | ✅ | Asset list with node IDs |
| Auto Layout | ✅ | Layout mode, padding, spacing |
| Flagged for LLM Review | ⚠️ Optional | Complex illustrations needing vision analysis |
| Inline Text Variations | ⚠️ Optional | characterStyleOverrides data |

## Stage 2 → Stage 3: Implementation Spec

**Producer:** design-analyst
**Consumer:** asset-manager

**File:** `docs/figma-reports/{file_key}-spec.md`

### Required Sections

| Section | Required | Description |
|---------|----------|-------------|
| Component Hierarchy | ✅ | Tree structure with component types |
| Design Tokens | ✅ | Mapped tokens (colors, typography, spacing) |
| Asset Requirements | ✅ | Asset list with types, formats, node IDs |
| Frame Properties | ✅ | Per-component dimension/style specs |
| Layer Order | ✅ | zIndex assignments |
| Flagged for LLM Review | ⚠️ Pass-through | Copied verbatim from validator |
| Image-with-Text | ⚠️ Optional | [contains-text] annotations |
| Edge-to-Edge | ⚠️ Optional | Edge-to-edge child markers |
| Glass Effects | ⚠️ Optional | Glass/translucent effect annotations |

## Stage 3 → Stage 4: Updated Spec + Assets

**Producer:** asset-manager
**Consumer:** code-generator-*

**File:** `docs/figma-reports/{file_key}-spec.md` (updated in-place)

### Added Sections

| Section | Required | Description |
|---------|----------|-------------|
| Downloaded Assets | ✅ | File paths, formats, dimensions |
| Asset Node Map | ✅ | node_id → local file path mapping |
| Flagged Frame Decisions | ⚠️ | DOWNLOAD_AS_IMAGE / GENERATE_AS_CODE |
| SVG Rendering Modes | ⚠️ | .original vs .template per icon |

## Stage 4 → Stage 5: Generated Code

**Producer:** code-generator-*
**Consumer:** compliance-checker

**File:** `docs/figma-reports/{file_key}-spec.md` (updated in-place)

### Added Sections

| Section | Required | Description |
|---------|----------|-------------|
| Generated Code | ✅ | File paths of generated components |
| Component File Map | ✅ | Component name → file path |
| Framework | ✅ | react / swiftui / vue / kotlin |
```

**Step 3: docs-index.md'ye ekle**

Core References tablosuna:
```markdown
| Pipeline Handoff | @skills/figma-to-code/references/pipeline-handoff.md | all agents |
```

**Step 4: Commit**

```bash
git add plugins/pb-figma/skills/figma-to-code/references/pipeline-handoff.md plugins/pb-figma/docs-index.md
git commit -m "docs(pipeline): create pipeline-handoff.md defining inter-agent data contracts"
```

---

### Task 15: Flagged Frames Workflow Netleştirmesi

**Files:**
- Modify: `agents/design-validator.md` (Flagged Frames output bölümü)
- Modify: `agents/design-analyst.md` (Flagged Frames pass-through bölümü)
- Modify: `agents/asset-manager.md` (Flagged Frames decision bölümü)

**Context:** Flagged Frames workflow'u 3 agent arasında dağıtık ve sahiplik belirsiz:
- Validator: Flags frames (5 trigger conditions)
- Analyst: Passes through verbatim (no decisions)
- Asset Manager: Makes DOWNLOAD_AS_IMAGE/GENERATE_AS_CODE decision via LLM Vision

Sorunlar:
1. Validator'ın flag formatı ile Analyst'in beklediği format arasında belirsizlik
2. Asset Manager'ın LLM Vision kararının spec'e geri yazılıp yazılmadığı belirsiz
3. Compliance checker'ın karar durumunu doğrulayıp doğrulayamadığı belirsiz

**Step 1: Üç agent'taki Flagged Frames bölümlerini oku**

- design-validator.md: Illustration complexity detection bölümü
- design-analyst.md: Flagged frames pass-through bölümü (satır 839-887)
- asset-manager.md: LLM Vision analysis bölümü (satır 221-244)

**Step 2: design-validator.md'ye flag format standardı ekle**

Illustration complexity detection bölümünün sonuna, output formatını netleştir:

```markdown
### Flagged Frames Output Format

When flagging frames for LLM review, use this exact format in the Validation Report:

```markdown
## Flagged for LLM Review

| Node ID | Node Name | Trigger | Dimensions | Details |
|---------|-----------|---------|------------|---------|
| 123:456 | chart-sales | Dark+Bright Siblings | 300×200 | Dark layer L=0.15, Bright layer L=0.72 |
| 789:012 | hero-illustration | High Vector Count | 400×300 | 47 vector paths |
```

**IMPORTANT:** Do NOT make decisions about these frames. Only flag them with evidence.
```

**Step 3: design-analyst.md pass-through bölümüne format doğrulaması ekle**

```markdown
### Flagged Frames Pass-Through

Copy the "Flagged for LLM Review" section **verbatim** from the Validation Report to the Implementation Spec. Do NOT:
- Interpret the flags
- Make DOWNLOAD_AS_IMAGE/GENERATE_AS_CODE decisions
- Add or remove entries
- Change the table format

**Verification:** The table in the spec must be identical to the table in the validation report.
```

**Step 4: asset-manager.md LLM Vision bölümüne karar kayıt formatı ekle**

```markdown
### Flagged Frame Decision Recording

After LLM Vision analysis, update the spec with decisions:

```markdown
## Flagged Frame Decisions

| Node ID | Node Name | Decision | Reasoning |
|---------|-----------|----------|-----------|
| 123:456 | chart-sales | DOWNLOAD_AS_IMAGE | Complex gradient overlay with data visualization |
| 789:012 | hero-illustration | GENERATE_AS_CODE | Simple vector composition, 3 distinct shapes |
```

**IMPORTANT:** This section must be added to the spec so compliance-checker can verify all flagged frames were resolved.
```

**Step 5: compliance-checker.md'ye flagged frame doğrulaması ekle**

```markdown
### Flagged Frame Resolution Check

Verify that:
- All entries from "Flagged for LLM Review" have corresponding entries in "Flagged Frame Decisions"
- Each decision is either DOWNLOAD_AS_IMAGE or GENERATE_AS_CODE
- DOWNLOAD_AS_IMAGE items have corresponding files in Downloaded Assets
- GENERATE_AS_CODE items have corresponding code in Generated Code
```

**Step 6: Commit**

```bash
git add plugins/pb-figma/agents/design-validator.md plugins/pb-figma/agents/design-analyst.md plugins/pb-figma/agents/asset-manager.md plugins/pb-figma/agents/compliance-checker.md
git commit -m "fix(pipeline): standardize Flagged Frames workflow across all agents"
```

---

## FAZ 5: Cross-Reference Tutarlılığı

### Task 16: Referans Dosyalarına "Used By" Header Standardı Ekle

**Files:**
- Modify: Tüm `skills/figma-to-code/references/*.md` dosyaları (30 dosya)

**Context:** Bazı referans dosyalarında "Used by" bilgisi var, bazılarında yok. Her referans dosyasının başına standart bir "Used by" satırı eklenecek.

**Step 1: Her referans dosyasının ilk 5 satırını oku**

Tüm 30 referans dosyasının başlangıcını oku. "Used by" veya "Used By" satırı olup olmadığını kontrol et.

**Step 2: Eksik olan dosyalara ekle**

Her referans dosyasının ilk heading'inden hemen sonra şu formatı ekle (yoksa):

```markdown
> **Used by:** [agent-list based on docs-index.md]
```

docs-index.md'deki "Used By" sütununa göre doğru agent listesini kullan.

Örnek:
- `color-extraction.md`: `> **Used by:** design-analyst, design-validator, code-generator-react, code-generator-swiftui, compliance-checker`
- `token-mapping.md`: `> **Used by:** code-generator-react, code-generator-swiftui, code-generator-kotlin`

**Step 3: docs-index.md ile tutarlılığı doğrula**

Her referans dosyasının "Used by" listesi, docs-index.md'deki "Used By" sütunuyla eşleşmeli.

Run: `grep -l "Used by" plugins/pb-figma/skills/figma-to-code/references/*.md | wc -l`
Expected: 30 (tüm referans dosyaları)

**Step 4: Commit**

```bash
git add plugins/pb-figma/skills/figma-to-code/references/*.md
git commit -m "fix(references): standardize 'Used by' headers across all reference files"
```

---

### Task 17: Agent Reference Loading Bölümlerini docs-index.md ile Senkronize Et

**Files:**
- Modify: `agents/code-generator-react.md` (Reference Loading bölümü)
- Modify: `agents/code-generator-swiftui.md` (Reference Loading bölümü)
- Modify: `agents/design-analyst.md` (Reference Loading bölümü)
- Modify: `agents/design-validator.md` (Reference Loading bölümü)
- Modify: `agents/compliance-checker.md` (Reference Loading bölümü)

**Context:** Agent'ların Reference Loading bölümünde listelenen referanslar, docs-index.md'deki "Used By" sütunuyla tam eşleşmiyor. Eksik referanslar eklenmeli.

**Step 1: docs-index.md'den her agent'ın kullanması gereken referansları çıkar**

docs-index.md Design Knowledge References tablosundaki "Used By" sütununu oku. Her agent için gerekli referans listesini çıkar.

**Step 2: Her agent'ın Reference Loading bölümünü güncelle**

Eksik referansları ekle. Örneğin design-validator şu anda sadece 2 referans listelerken, docs-index.md'ye göre şunları da kullanıyor:
- frame-properties.md
- color-extraction.md
- opacity-extraction.md
- illustration-detection.md
- font-handling.md

Bu eksik olanları Reference Loading bölümüne ekle.

**Step 3: Senkronizasyonu doğrula**

Her agent dosyasında listelenen referans sayısını say ve docs-index.md ile karşılaştır.

**Step 4: Commit**

```bash
git add plugins/pb-figma/agents/*.md
git commit -m "fix(agents): sync Reference Loading sections with docs-index.md"
```

---

## FAZ 6: Pipeline Yapılandırılabilirlik & Checkpoint

### Task 18: Pipeline Konfigürasyon Referansı Oluştur

**Files:**
- Create: `skills/figma-to-code/references/pipeline-config.md`
- Modify: `docs-index.md`

**Context:** Pipeline şu anda hardcoded değerler kullanıyor (breakpoints: 375/768/1440, visual match threshold: 95%, retry count: 3, batch size: 10). Bunlar konfigüre edilebilir olmalı.

**Step 1: Tüm agent'lardaki hardcoded değerleri listele**

Explore agent'lardan gelen bulgulara göre:
- Breakpoints: 375px, 768px, 1440px (responsive-patterns.md, compliance-checker.md)
- Visual match threshold: ≥95% PASS, 85-94% WARN (compliance-checker.md)
- Retry count: 3 (error-recovery.md)
- Asset batch size: 10 (asset-manager.md)
- Rate limit delay: 2s initial, exponential backoff (code-generator-base.md)
- Icon size threshold: ≤48px icon, >50px illustration (illustration-detection.md)
- Touch target minimum: 44x44px (compliance-checker.md)

**Step 2: pipeline-config.md oluştur**

```markdown
# Pipeline Configuration Reference

> **Used by:** All pipeline agents

## Overview

This document defines configurable values used across the pipeline. When the skill file or user specifies overrides, agents should use those values instead of defaults.

## Default Configuration

### Responsive Breakpoints

| Name | Width | Usage |
|------|-------|-------|
| mobile | 375px | Primary mobile viewport |
| tablet | 768px | Tablet/iPad viewport |
| desktop | 1440px | Desktop viewport |

### Visual Validation

| Setting | Default | Description |
|---------|---------|-------------|
| pass_threshold | 95% | Minimum visual match % for PASS |
| warn_threshold | 85% | Minimum visual match % for WARN |
| screenshot_scale | 2x | Figma screenshot scale factor |

### Asset Processing

| Setting | Default | Description |
|---------|---------|-------------|
| batch_size | 10 | Assets per API call |
| retry_count | 3 | Max retries per failed operation |
| retry_base_delay | 1s | Initial retry delay (exponential backoff) |
| rate_limit_delay | 2s | Delay between MCP calls |

### Asset Classification

| Setting | Default | Description |
|---------|---------|-------------|
| icon_max_size | 48px | Max dimension for icon classification |
| illustration_min_size | 50px | Min dimension for illustration classification |
| vector_complexity_threshold | 10 | Vector paths triggering complexity review |

### Accessibility

| Setting | Default | Description |
|---------|---------|-------------|
| min_touch_target | 44px | Minimum touch target size (mobile) |
| contrast_ratio_normal | 4.5 | WCAG AA contrast ratio for normal text |
| contrast_ratio_large | 3.0 | WCAG AA contrast ratio for large text |

## Overriding Defaults

Agents should check the skill invocation prompt for configuration overrides. Format:

```
Task(subagent_type="pb-figma:compliance-checker",
     prompt="Validate... Config: { pass_threshold: 90%, breakpoints: [360, 768, 1280] }")
```

If no overrides specified, use defaults from this document.
```

**Step 3: docs-index.md'ye ekle**

Core References tablosuna:
```markdown
| Pipeline Config | @skills/figma-to-code/references/pipeline-config.md | all agents |
```

**Step 4: Commit**

```bash
git add plugins/pb-figma/skills/figma-to-code/references/pipeline-config.md plugins/pb-figma/docs-index.md
git commit -m "docs(pipeline): create pipeline-config.md with configurable defaults"
```

---

### Task 19: Checkpoint Sistemini error-recovery.md'de Netleştir

**Files:**
- Modify: `skills/figma-to-code/references/error-recovery.md`

**Context:** error-recovery.md checkpoint sistemi tanımlıyor (`.qa/checkpoint-{N}.json`) ama bu dosyalar hiçbir agent tarafından üretilmiyor. Ya kaldırılmalı ya da agent'lara entegre edilmeli.

**Step 1: error-recovery.md'deki checkpoint bölümünü oku**

Checkpoint ile ilgili satırları bul ve oku.

**Step 2: Checkpoint bölümünü "Planned Feature" olarak işaretle**

Mevcut checkpoint bölümünü şu şekilde güncelle:

```markdown
## Checkpoint System (Planned)

> ⚠️ **Status:** This checkpoint system is documented as a future feature. Currently, pipeline state is passed through spec files (`{file_key}-spec.md`). Agents do not yet produce `.qa/checkpoint-*.json` files.

### Future Implementation

When implemented, save intermediate state after each phase:

```
.qa/checkpoint-1-validation.json    (Phase 1)
.qa/checkpoint-2-spec.json          (Phase 2)
.qa/checkpoint-3-assets.json        (Phase 3)
.qa/checkpoint-4-code.json          (Phase 4)
```

### Current Recovery Mechanism

Pipeline currently recovers by:
1. Re-reading the spec file from `docs/figma-reports/`
2. Checking which sections are already populated
3. Resuming from the last completed phase
```

**Step 3: Commit**

```bash
git add plugins/pb-figma/skills/figma-to-code/references/error-recovery.md
git commit -m "docs(error-recovery): clarify checkpoint system as planned feature"
```

---

## FAZ 7: Eksik Pipeline Aşamaları & Dokümentasyon

### Task 20: Unused Prompt Templates Temizliği

**Files:**
- Delete or archive: `skills/figma-to-code/assets/prompts/` (if exists)
- Modify: `docs-index.md`

**Context:** docs-index.md'de 5 adet "⚠️ Unused" prompt template listelenmiş (analyze-design.md, mapping-planning.md, generate-component.md, validate-refine.md, handoff.md). Bu dosyalar aktif agent'lar tarafından kullanılmıyor.

**Step 1: Prompt template dosyalarının varlığını kontrol et**

Run: `ls plugins/pb-figma/skills/figma-to-code/assets/prompts/ 2>/dev/null`

**Step 2: docs-index.md'deki Prompt Templates bölümünü güncelle**

Eğer dosyalar mevcutsa, bölümü şu şekilde güncelle:

```markdown
## Prompt Templates (Deprecated)

> **⚠️ Deprecated:** These prompt templates were used in versions prior to v1.0. Active agents now load references directly. These files are kept for historical reference only and may be removed in a future version.

| Template | Original Purpose | Status |
|----------|------------------|--------|
| analyze-design.md | Design analysis prompts | ⚠️ Deprecated |
| mapping-planning.md | Mapping & planning prompts | ⚠️ Deprecated |
| generate-component.md | Component generation prompts | ⚠️ Deprecated |
| validate-refine.md | Validation prompts | ⚠️ Deprecated |
| handoff.md | Handoff documentation | ⚠️ Deprecated |
```

**Step 3: Commit**

```bash
git add plugins/pb-figma/docs-index.md
git commit -m "docs(docs-index): mark prompt templates as deprecated"
```

---

### Task 21: CI/CD Integration Referansını Doğru İşaretle

**Files:**
- Modify: `docs-index.md`

**Context:** docs-index.md'de CI/CD Integration `⚠️ Not integrated (no agent uses this)` olarak işaretlenmiş. Bu doğru bir status ama daha net olmalı.

**Step 1: CI/CD bölümünü güncelle**

```markdown
### CI/CD & Integration (Not Yet Integrated)

> **Status:** These references document future integration points. No agent currently uses them.

| Topic | Path | Used By |
|-------|------|---------|
| Storybook Integration | @skills/figma-to-code/references/storybook-integration.md | code-generator-react (future) |
| CI/CD Integration | @skills/figma-to-code/references/ci-cd-integration.md | Pipeline orchestration (future) |
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/docs-index.md
git commit -m "docs(docs-index): clarify CI/CD integration status"
```

---

### Task 22: Missing Pipeline Stages Dokümanı Oluştur

**Files:**
- Create: `skills/figma-to-code/references/planned-features.md`
- Modify: `docs-index.md`

**Context:** Birçok pipeline aşaması eksik: animation/transition, theme/dark mode, design system extraction, component variant completeness. Bunları planlı özellikler olarak belgelemek gelecekteki geliştirmelere yol gösterecek.

**Step 1: planned-features.md oluştur**

```markdown
# Planned Features & Missing Pipeline Stages

> **Status:** These features are identified as gaps in the current pipeline. They are documented here for future implementation planning.

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
```

**Step 2: docs-index.md'ye ekle**

```markdown
| Planned Features | @skills/figma-to-code/references/planned-features.md | Planning reference |
```

**Step 3: Commit**

```bash
git add plugins/pb-figma/skills/figma-to-code/references/planned-features.md plugins/pb-figma/docs-index.md
git commit -m "docs(pipeline): create planned-features.md documenting missing stages"
```

---

### Task 23: Final Doğrulama ve Satır Sayısı Raporu

**Files:**
- Read: Tüm değiştirilen dosyalar

**Step 1: Agent dosya boyutlarını kontrol et**

Run: `wc -l plugins/pb-figma/agents/*.md`
Expected:
- code-generator-react.md: ~740 satır (was 1,341 — ~45% azalma)
- code-generator-swiftui.md: ~1,100 satır (was 1,952 — ~44% azalma)
- Diğerleri: Minimal değişiklik

**Step 2: Referans dosya sayısını kontrol et**

Run: `ls plugins/pb-figma/skills/figma-to-code/references/*.md | wc -l`
Expected: 34 (was 30 + react-patterns.md + swiftui-patterns.md + pipeline-handoff.md + pipeline-config.md + planned-features.md = 35, but some may not be new)

**Step 3: Hiçbir kırık referans olmadığını doğrula**

Run: `grep -r "Glob(" plugins/pb-figma/agents/*.md | grep -oP 'references/[^"]+\.md' | sort -u`

Her bulunan pattern için dosyanın var olduğunu doğrula.

**Step 4: docs-index.md'nin tutarlı olduğunu doğrula**

docs-index.md'deki tüm referans path'lerinin gerçek dosyalara karşılık geldiğini kontrol et.

**Step 5: Commit (eğer düzeltme gerekiyorsa)**

```bash
git add -A plugins/pb-figma/
git commit -m "fix(pb-figma): final validation fixes after comprehensive improvements"
```

---

## FAZ 8: @skills Inline Reference Cleanup ✅

> **Status:** ✅ Completed. 36 inline @skills refs replaced across 7 agent files with Glob-based format.
> **Context:** 36 inline `@skills/figma-to-code/references/...` references remain in agent bodies despite the "Do NOT use @skills paths" warning in every agent's Reference Loading section.

### Task 24: Replace inline @skills references with Glob-based format

**Files (7 agents, 36 inline references):**
- `agents/design-analyst.md` — 11 inline refs (lines: 150, 196, 239, 417, 448, 476, 497, 544, 555, 907, 1178)
- `agents/design-validator.md` — 7 inline refs (lines: 134, 157, 167, 190, 285, 450, 501)
- `agents/code-generator-swiftui.md` — 7 inline refs (lines: 112, 202, 219, 569, 591, 603, 884)
- `agents/code-generator-react.md` — 6 inline refs (lines: 69, 86, 398, 464, 504, 807)
- `agents/compliance-checker.md` — 8 inline refs (lines: 133, 134, 173, 226, 339, 550, 612, 613)
- `agents/font-manager.md` — 2 inline refs (lines: 82, 242)
- `agents/asset-manager.md` — 2 inline refs (lines: 173, 470)

**Transform pattern:**
```
BEFORE:
> **Reference:** @skills/figma-to-code/references/{filename}.md — Description text

AFTER:
> **Reference:** `{filename}.md` — Description text
> Load via: `Glob("**/references/{filename}.md")` → `Read()`
```

For comment-style refs:
```
BEFORE:
    // ↑ See @skills/figma-to-code/references/text-decoration.md for decoration value mapping

AFTER:
    // ↑ See references/text-decoration.md for decoration value mapping (load via Glob)
```

**Commit:** `fix(agents): replace 36 inline @skills paths with Glob-based references`

---

## FAZ 9: API Call Optimization ✅

> **Status:** ✅ Completed. Validation report enriched with coordinates, textAutoResize, gradient details. Cache-first guidance added to design-analyst and asset-manager.
> **Context:** Pipeline makes ~37 `figma_get_node_details` calls across agents. Design-validator already fetches most data but outputs summaries. Design-analyst re-queries the same nodes for properties already in the validation report. Target: eliminate 70%+ of redundant analyst calls.

### Task 25: Expand design-validator output with missing properties

**Modify:** `agents/design-validator.md`

**Add to Frame Properties table output (around line 143):**
- `absoluteBoundingBox.x` and `absoluteBoundingBox.y` columns — enables layer order z-index calculation without re-querying
- Section instruction: "Include absolute coordinates for ALL frames to enable downstream layer-order calculation"

**Add new "Text Properties" section to validation report output:**
- `textAutoResize` property for each text node (NONE / WIDTH_AND_HEIGHT / HEIGHT)
- This eliminates design-analyst's need to query `figma_get_node_details` for text sizing

**Add "Gradient Details" subsection to Design Tokens:**
- When gradient fills detected, include: gradient type, stop positions, colors, opacity
- This eliminates design-analyst's separate gradient extraction calls

**Commit:** `feat(design-validator): expand report with coordinates, textAutoResize, gradient details`

### Task 26: Add cache-first guidance to design-analyst

**Modify:** `agents/design-analyst.md`

**Add new section after "Reference Loading" (around line 25):**
```markdown
## Data Source Priority

When extracting design properties, ALWAYS follow this order:
1. **Read from Validation Report FIRST** — Frame Properties table, Auto Layout table, Design Tokens, Assets Inventory contain pre-extracted data
2. **Only call Figma MCP if data is MISSING** from the validation report
3. **Never re-query** for: frame dimensions, corner radius, borders, card icon node IDs, character style overrides, text decorations — these are already in the validation report

This reduces API calls by ~70% and speeds up the pipeline.
```

**Add specific guidance in existing sections:**
- Frame Properties parsing (line ~239): Add "Read from validation report Frame Properties table. Only call `figma_get_node_details` if height/coordinates are missing."
- Card icon classification (line ~229): Add "Read from validation report Assets Inventory. Icon node IDs and types are pre-classified."
- Text decoration (line ~557): Add "Check validation report Inline Text Variations table. Only call MCP if textDecoration property not included."

**Commit:** `perf(design-analyst): add cache-first data source priority to reduce redundant API calls`

### Task 27: Add cache-first guidance to asset-manager

**Modify:** `agents/asset-manager.md`

**Add similar "Data Source Priority" section:**
- Read node IDs and asset types from spec (which came from validation report)
- Only call `figma_get_node_details` for nodes not already described in spec
- Batch `figma_export_assets` calls instead of one-by-one

**Commit:** `perf(asset-manager): add cache-first guidance for API call reduction`

---

## FAZ 10: Pipeline Parallelization (Conservative) ✅

> **Status:** ✅ Completed. SKILL.md updated with parallel fork diagram and run_in_background=True for font-manager.
> **Context:** Currently all 5 pipeline stages run sequentially. Asset-manager and font-manager are independent after design-analyst completes — they can run in parallel. Conservative approach: only parallelize these two.

### Task 28: Update SKILL.md invocation sequence for parallel execution

**Modify:** `skills/figma-to-code/SKILL.md` (lines 61-83)

**Replace sequential Steps 3-4 with parallel dispatch:**
```python
# Steps 1-2: Sequential (unchanged)
# Step 1: Design Validator
Task(subagent_type="pb-figma:design-validator",
     prompt="Validate Figma URL: {url}")

# Step 2: Design Analyst (MANDATORY - creates Implementation Spec)
Task(subagent_type="pb-figma:design-analyst",
     prompt="Create Implementation Spec from: docs/figma-reports/{file_key}-validation.md")

# Step 3: Asset Manager + Font Manager (PARALLEL)
# Launch BOTH in a single message with multiple Task calls:
Task(subagent_type="pb-figma:asset-manager",
     prompt="Download assets from spec: docs/figma-reports/{file_key}-spec.md")
Task(subagent_type="pb-figma:font-manager",
     prompt="Detect and setup fonts from spec: docs/figma-reports/{file_key}-spec.md",
     run_in_background=True)

# Step 4: Code Generator (after asset-manager completes; font-manager continues in background)
Task(subagent_type="pb-figma:code-generator-{framework}",
     prompt="Generate code from spec: docs/figma-reports/{file_key}-spec.md")

# Step 5: Compliance Checker
Task(subagent_type="pb-figma:compliance-checker",
     prompt="Validate implementation against spec: docs/figma-reports/{file_key}-spec.md")
```

**Update pipeline diagram** to show parallel fork:
```
design-analyst
    |
    +---> asset-manager ----+
    |                       |
    +---> font-manager      |  (background)
         (parallel)         |
                            v
                    code-generator
                            |
                            v
                    compliance-checker
```

**Commit:** `feat(pipeline): parallelize asset-manager and font-manager after design-analyst`

---

## FAZ 11: Framework Detection Standardization ✅

> **Status:** ✅ Completed. framework-detection.md reference created, docs-index + SKILL.md + 2 code-generator agents updated.
> **Context:** React and SwiftUI agents use completely different detection logic. No shared structure exists. A common reference ensures consistent detection flow.

### Task 29: Create framework-detection.md reference

**Create:** `skills/figma-to-code/references/framework-detection.md`

**Content structure:**
```markdown
# Framework Detection Reference

## Standard Detection Flow
1. Scan project root for framework indicators
2. Confirm with user if ambiguous
3. Map to MCP code generation parameter

## Detection Matrix

| Indicator | Framework | MCP Parameter | Agent |
|-----------|-----------|---------------|-------|
| *.xcodeproj, *.xcworkspace, Package.swift | SwiftUI | `swiftui` | code-generator-swiftui |
| package.json with "react" or "next" | React/Next.js | `react_tailwind` | code-generator-react |
| package.json with "vue" or "nuxt" | Vue/Nuxt | `vue_tailwind` | code-generator-vue 🚧 |
| build.gradle.kts with "androidx.compose" | Kotlin Compose | `kotlin` | code-generator-kotlin 🚧 |
| None detected | React (default) | `react_tailwind` | code-generator-react |

## Variant Detection

### React Variants
- Next.js: `next.config.*` present OR "next" in package.json
- Vite: `vite.config.*` present
- CRA: `react-scripts` in package.json

### SwiftUI Variants
- Xcode Project: `*.xcodeproj` present
- Xcode Workspace: `*.xcworkspace` present
- Swift Package: `Package.swift` without xcodeproj

### Tailwind Detection (for web frameworks)
- `tailwind.config.*` OR `tailwindcss` in package.json dependencies
- If missing: warn user, suggest adding Tailwind

## User Confirmation
When detection is ambiguous (e.g., both React and Vue dependencies):
- Present detected options via AskUserQuestion
- Default to the first match in the Detection Matrix
```

**Update agents to reference this file:**
- `agents/code-generator-react.md`: Add `framework-detection.md` to Reference Loading section
- `agents/code-generator-swiftui.md`: Add `framework-detection.md` to Reference Loading section
- `SKILL.md`: Add to references table

**Update docs-index.md:** Add framework-detection.md entry

**Commit:** `feat(references): add framework-detection.md for standardized detection across agents`

---

## FAZ 12: Code Connect MCP Integration ✅

> **Status:** ✅ Completed. design-analyst gets lookup step, both code-generators get registration step + new MCP tools.
> **Context:** `figma_get_code_connect_map` and `figma_add_code_connect_map` MCP tools are available but 0% used. These tools map Figma components to existing codebase components — critical for reuse.

### Task 30: Add Code Connect lookup to design-analyst

**Modify:** `agents/design-analyst.md`

**Add new step in component analysis workflow (around line 140):**
```markdown
### Step 2b: Check Code Connect Mappings

Before creating Implementation Spec component entries:
1. Call `figma_get_code_connect_map(file_key)` to get existing component mappings
2. For each mapped component:
   - Add `code_connect: true` flag to spec entry
   - Include `component_path` and `component_name` from mapping
   - Include `props_mapping` for property translation
3. Mark unmapped components as `code_connect: false`

This enables code generators to REUSE existing components instead of generating from scratch.
```

**Commit:** `feat(design-analyst): integrate Code Connect MCP for component reuse detection`

### Task 31: Add Code Connect registration to code generators

**Modify:** `agents/code-generator-react.md` and `agents/code-generator-swiftui.md`

**Add post-generation step:**
```markdown
### Step 7: Register Code Connect Mapping

After generating each component:
1. Check if component already has Code Connect mapping (`code_connect: true` in spec)
2. If NOT mapped, register new mapping:
   ```
   figma_add_code_connect_map(
     file_key="{file_key}",
     node_id="{component_node_id}",
     component_path="{generated_file_path}",
     component_name="{ComponentName}",
     props_mapping={figma_prop: code_prop}
   )
   ```
3. This enables future pipeline runs to reuse this component
```

**Update code-connect-guide.md reference:** Add pipeline integration examples

**Commit:** `feat(code-generators): register Code Connect mappings after component generation`

---

## FAZ 13: Additional SwiftUI Prompt Bloat Reduction ✅

> **Status:** ✅ Completed. 4 sections extracted (287 lines removed), code-generator-swiftui.md: 1348→1061 lines.
> **Context:** code-generator-swiftui.md is still 1,347 lines (target was ~1,100). Several extractable sections remain.

### Task 32: Extract Selective Padding patterns to reference

**Modify:** `agents/code-generator-swiftui.md` (lines 370-419, ~50 lines)
**Target:** `skills/figma-to-code/references/swiftui-patterns.md` (append)

Extract the "Selective Padding (Edge-to-Edge Children)" section including:
- Edge-to-edge detection logic
- ClipContent pattern
- WRONG vs CORRECT padding examples

Replace with Glob reference pointer.

### Task 33: Extract Component Example to reference

**Modify:** `agents/code-generator-swiftui.md` (lines 1016-1148, ~133 lines)
**Create:** `skills/figma-to-code/references/swiftui-component-example.md`

Extract the complete CardView example including:
- MARK comments structure
- DocC documentation
- Property wrappers
- Accessibility modifiers
- Computed properties
- #Preview macro

Replace with Glob reference pointer.

### Task 34: Extract Inline Text Variations to reference

**Modify:** `agents/code-generator-swiftui.md` (lines 706-810, ~105 lines)
**Create:** `skills/figma-to-code/references/inline-text-variations.md`

Extract:
- Multi-color text generation with .foregroundStyle()
- Single vs multi-variation detection
- iOS version guards for text variations
- Code examples for both patterns

Replace with Glob reference pointer.

### Task 35: Extract Manual Generation Fallback to reference

**Modify:** `agents/code-generator-swiftui.md` (lines 1210-1255, ~46 lines)
**Target:** `skills/figma-to-code/references/swiftui-patterns.md` (append)

Extract the manual struct generation fallback pattern.

**Expected reduction:** ~334 lines (from 1,347 to ~1,013)

**Commit:** `refactor(code-generator-swiftui): extract 4 sections to references (~334 lines)`

---

## Özet

| Faz | Task Sayısı | Tahmini Etki |
|-----|-------------|-------------|
| Faz 1: Path Düzeltmeleri | 4 task | Tüm path tutarsızlıkları giderilir |
| Faz 2: Dil Standardizasyonu | 2 task | TR/EN karışıklığı giderilir |
| Faz 3: Prompt Bloat Azaltma | 7 task | ~1,500+ satır agent dosyalarından referanslara taşınır |
| Faz 4: Handoff Standardizasyonu | 2 task | Agent arası veri kontratı netleşir |
| Faz 5: Cross-Reference Tutarlılığı | 2 task | Tüm referanslar senkronize edilir |
| Faz 6: Yapılandırılabilirlik | 2 task | Hardcoded değerler konfigüre edilebilir olur |
| Faz 7: Eksik Aşamalar | 4 task | Gelecek geliştirmeler belgelenir |
| **Faz 8: @skills Cleanup** | **1 task** | **36 inline @skills referansı Glob formatına çevrilir** |
| **Faz 9: API Optimizasyonu** | **3 task** | **~70% redundant API call azaltma** |
| **Faz 10: Parallelization** | **1 task** | **asset-manager + font-manager paralel çalışır** |
| **Faz 11: Framework Detection** | **1 task** | **Standardize detection across agents** |
| **Faz 12: Code Connect MCP** | **2 task** | **Component reuse via MCP integration** |
| **Faz 13: SwiftUI Bloat** | **4 task** | **~334 satır daha extract edilir** |
| **TOPLAM** | **35 task** | **Pipeline token-efficient, parallel, reuse-capable** |
