# Code Connect Workflow Guide

> **Used by:** design-analyst

Figma component'lerini mevcut codebase component'lerine eşleştirmek için kapsamlı kılavuz.

---

## Code Connect Nedir?

Code Connect, Figma tasarım component'leri ile kod implementasyonları arasında köprü kuran bir sistemdir. Bu eşleştirmeler sayesinde:

### Faydaları

| Fayda | Açıklama |
|-------|----------|
| **Mevcut Component Kullanımı** | Yeni kod üretmek yerine var olan component'leri import eder |
| **Duplicate Kod Önleme** | Aynı component için tekrar tekrar kod üretilmez |
| **Tasarım-Kod Senkronizasyonu** | Figma'daki değişiklikler doğru component'e yönlendirilir |
| **Props Eşleştirmesi** | Figma variant'ları otomatik olarak kod props'larına çevrilir |
| **Tutarlılık** | Tüm ekipte aynı component'lerin kullanılmasını sağlar |

### Ne Zaman Kullanılır?

```
✅ Mevcut design system component'leri varsa
✅ Tekrar kullanılabilir UI kit kullanılıyorsa
✅ Figma'da component library tanımlıysa
✅ Kod tutarlılığı kritik ise

❌ Sıfırdan proje başlatılıyorsa
❌ One-off tasarımlar için
❌ Prototip/exploration aşamasında
```

---

## MCP Tools

Code Connect için 3 temel MCP aracı bulunur.

### 1. figma_get_code_connect_map

Mevcut eşleştirmeleri getirir.

**Parameters:**
```typescript
{
  params: {
    file_key: string,      // Figma dosya anahtarı
    node_id?: string       // Opsiyonel - belirli node için
  }
}
```

**Kullanım:**
```javascript
// Tüm eşleştirmeleri getir
mcp__pixelbyte-figma-mcp__figma_get_code_connect_map({
  params: {
    file_key: "ABC123xyz"
  }
})

// Belirli node için eşleştirme getir
mcp__pixelbyte-figma-mcp__figma_get_code_connect_map({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456"
  }
})
```

**Dönen Veri:**
```json
{
  "mappings": [
    {
      "node_id": "123:456",
      "component_path": "src/components/ui/button.tsx",
      "component_name": "Button",
      "props_mapping": {
        "Variant": "variant",
        "Size": "size"
      },
      "variants": {
        "Primary": { "variant": "default" },
        "Secondary": { "variant": "secondary" },
        "Destructive": { "variant": "destructive" }
      },
      "example": "<Button variant=\"default\" size=\"md\">Click me</Button>"
    }
  ]
}
```

---

### 2. figma_add_code_connect_map

Yeni eşleştirme ekler veya mevcut olanı günceller.

**Parameters:**
```typescript
{
  params: {
    file_key: string,           // Figma dosya anahtarı
    node_id: string,            // Figma node ID
    component_path: string,     // Kod dosya yolu
    component_name: string,     // Component adı
    props_mapping?: object,     // Figma prop → Kod prop eşleştirmesi
    variants?: object,          // Variant değer eşleştirmeleri
    example?: string            // Örnek kullanım kodu
  }
}
```

**Kullanım:**
```javascript
// Button component eşleştirmesi
mcp__pixelbyte-figma-mcp__figma_add_code_connect_map({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456",
    component_path: "src/components/ui/button.tsx",
    component_name: "Button",
    props_mapping: {
      "Variant": "variant",
      "Size": "size",
      "Disabled": "disabled"
    },
    variants: {
      "Primary": { "variant": "default" },
      "Secondary": { "variant": "secondary" },
      "Ghost": { "variant": "ghost" },
      "Small": { "size": "sm" },
      "Medium": { "size": "md" },
      "Large": { "size": "lg" }
    },
    example: "<Button variant=\"default\" size=\"md\">Label</Button>"
  }
})

// Card component eşleştirmesi
mcp__pixelbyte-figma-mcp__figma_add_code_connect_map({
  params: {
    file_key: "ABC123xyz",
    node_id: "789:012",
    component_path: "src/components/ui/card.tsx",
    component_name: "Card",
    props_mapping: {
      "Has Header": "withHeader",
      "Has Footer": "withFooter"
    },
    variants: {
      "With Shadow": { "className": "shadow-lg" },
      "Flat": { "className": "border" }
    },
    example: "<Card withHeader><CardHeader>Title</CardHeader><CardContent>Content</CardContent></Card>"
  }
})
```

---

### 3. figma_remove_code_connect_map

Mevcut eşleştirmeyi kaldırır.

**Parameters:**
```typescript
{
  params: {
    file_key: string,    // Figma dosya anahtarı
    node_id: string      // Kaldırılacak node ID
  }
}
```

**Kullanım:**
```javascript
mcp__pixelbyte-figma-mcp__figma_remove_code_connect_map({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456"
  }
})
```

**Ne Zaman Kullanılır:**
- Component deprecated olduğunda
- Yanlış eşleştirme yapıldığında
- Component yeniden adlandırıldığında (önce sil, sonra yeni ekle)

---

## Workflow (5 Aşama)

### Phase 1: Component Inventory (Codebase Analizi)

Mevcut codebase'deki component'leri listele.

**Amaç:** Hangi component'lerin zaten var olduğunu belirle.

**Komutlar:**
```bash
# UI component'lerini bul
find src/components/ui -name "*.tsx" -type f | head -20

# Tüm export edilen component'leri listele
grep -r "export.*function\|export.*const" src/components/ui --include="*.tsx" | head -30

# shadcn/ui component'lerini kontrol et
ls -la src/components/ui/

# Feature component'lerini bul
find src/features -name "*.tsx" -path "*/components/*" | head -20
```

**Çıktı Formatı:**
```
Component Inventory
==================

UI Components (src/components/ui/):
- button.tsx         → Button, buttonVariants
- card.tsx           → Card, CardHeader, CardContent, CardFooter
- input.tsx          → Input
- dialog.tsx         → Dialog, DialogTrigger, DialogContent
- avatar.tsx         → Avatar, AvatarImage, AvatarFallback

Feature Components:
- src/features/auth/components/AuthModal.tsx
- src/features/video/components/VideoCard.tsx
- src/features/explore/components/ExploreCard.tsx
```

---

### Phase 2: Figma Component Analysis

Figma dosyasındaki component'leri MCP ile analiz et.

**Amaç:** Figma'daki component'leri ve variant'larını tespit et.

**Adımlar:**

```javascript
// 1. Dosya yapısını al
mcp__pixelbyte-figma-mcp__figma_get_file_structure({
  params: {
    file_key: "ABC123xyz",
    depth: 3,
    response_format: "markdown"
  }
})

// 2. Component detaylarını al
mcp__pixelbyte-figma-mcp__figma_get_node_details({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456",  // Component node ID
    response_format: "json"
  }
})
```

**Beklenen Çıktı:**
```
Figma Components
================

Page: Design System
├── Components
│   ├── Button (123:456)
│   │   ├── Variants: Primary, Secondary, Ghost, Destructive
│   │   ├── Sizes: Small, Medium, Large
│   │   └── States: Default, Hover, Disabled
│   ├── Card (789:012)
│   │   ├── Variants: Default, Elevated
│   │   └── Has: Header, Content, Footer
│   └── Input (345:678)
│       ├── Types: Text, Password, Email
│       └── States: Default, Focus, Error, Disabled
```

---

### Phase 3: Matching (Eşleştirme)

Figma component'lerini codebase component'leriyle eşleştir.

**Match Score Kriterleri:**

| Kriter | Puan | Açıklama |
|--------|------|----------|
| İsim Eşleşmesi | +40 | Figma ve kod isimleri aynı/benzer |
| Variant Uyumu | +25 | Variant'lar props'larla eşleşiyor |
| Yapısal Benzerlik | +20 | Child element'ler uyumlu |
| Props Kapsamı | +15 | Tüm Figma özellikleri karşılanıyor |

**Eşleştirme Tablosu Örneği:**

| Figma Component | Codebase Component | Match Score | Notlar |
|-----------------|-------------------|-------------|--------|
| Button (123:456) | `ui/button.tsx` → Button | 95% | Variant'lar tam uyumlu |
| Card (789:012) | `ui/card.tsx` → Card | 85% | Footer optional kontrolü eksik |
| Input (345:678) | `ui/input.tsx` → Input | 90% | Error state eklenmeli |
| Avatar (111:222) | `ui/avatar.tsx` → Avatar | 100% | Tam uyum |
| VideoCard (333:444) | `features/video/VideoCard.tsx` | 75% | Bazı props farklı |
| HeroSection (555:666) | ❌ Yok | 0% | Yeni oluşturulmalı |

**Karar Matrisi:**

```
Match Score ≥ 90%  → Direkt eşleştir
Match Score 70-89% → Eşleştir + Props mapping ayarla
Match Score 50-69% → Wrapper component düşün
Match Score < 50%  → Yeni component üret
```

---

### Phase 4: Props Mapping

Figma variant'larını kod props'larına çevir.

**Mapping Stratejileri:**

#### 1. Direkt Mapping (1:1)

Figma property adı → Kod prop adı

```javascript
props_mapping: {
  "Variant": "variant",      // Figma "Variant" → kod "variant"
  "Size": "size",            // Figma "Size" → kod "size"
  "Disabled": "disabled"     // Figma "Disabled" → kod "disabled"
}
```

#### 2. Değer Dönüştürme

Figma değerleri → Kod değerleri

```javascript
variants: {
  // Figma variant adı → Kod prop değerleri
  "Primary": { "variant": "default" },      // Figma "Primary" → variant="default"
  "Secondary": { "variant": "secondary" },
  "Destructive": { "variant": "destructive" },
  "Ghost": { "variant": "ghost" },
  "Small": { "size": "sm" },
  "Medium": { "size": "md" },
  "Large": { "size": "lg" }
}
```

#### 3. Kompozit Mapping

Birden fazla Figma property → Tek kod prop

```javascript
// Figma: Type=Icon, Position=Left
// Kod: iconPosition="left"

variants: {
  "Icon Left": { "iconPosition": "left" },
  "Icon Right": { "iconPosition": "right" },
  "No Icon": { "iconPosition": undefined }
}
```

#### 4. Boolean Mapping

Figma toggle → Kod boolean

```javascript
props_mapping: {
  "Show Icon": "showIcon",
  "Has Badge": "hasBadge",
  "Is Loading": "isLoading"
}

// Figma'da "Show Icon=true" → showIcon={true}
```

**Tam Props Mapping Örneği:**

```javascript
// Button Component için kapsamlı mapping
mcp__pixelbyte-figma-mcp__figma_add_code_connect_map({
  params: {
    file_key: "ABC123xyz",
    node_id: "123:456",
    component_path: "src/components/ui/button.tsx",
    component_name: "Button",
    props_mapping: {
      "Variant": "variant",
      "Size": "size",
      "Disabled": "disabled",
      "Loading": "isLoading",
      "Icon Position": "iconPosition"
    },
    variants: {
      // Variant değerleri
      "Primary": { "variant": "default" },
      "Secondary": { "variant": "secondary" },
      "Outline": { "variant": "outline" },
      "Ghost": { "variant": "ghost" },
      "Link": { "variant": "link" },
      "Destructive": { "variant": "destructive" },
      // Size değerleri
      "Small": { "size": "sm" },
      "Medium": { "size": "default" },
      "Large": { "size": "lg" },
      "Icon Only": { "size": "icon" },
      // Icon pozisyonları
      "Icon Left": { "iconPosition": "left" },
      "Icon Right": { "iconPosition": "right" }
    },
    example: `<Button
  variant="default"
  size="md"
  disabled={false}
  isLoading={false}
>
  Click me
</Button>`
  }
})
```

---

### Phase 5: Register Mappings

Tüm eşleştirmeleri kaydet.

**Toplu Kayıt Scripti:**

```javascript
// Tüm UI component'leri için eşleştirmeleri kaydet
const mappings = [
  {
    node_id: "123:456",
    component_path: "src/components/ui/button.tsx",
    component_name: "Button",
    props_mapping: { "Variant": "variant", "Size": "size" },
    variants: { "Primary": { "variant": "default" } }
  },
  {
    node_id: "789:012",
    component_path: "src/components/ui/card.tsx",
    component_name: "Card",
    props_mapping: {},
    variants: {}
  },
  {
    node_id: "345:678",
    component_path: "src/components/ui/input.tsx",
    component_name: "Input",
    props_mapping: { "Type": "type", "Error": "error" },
    variants: { "Error": { "error": true } }
  }
];

// Her bir mapping için kayıt yap
for (const mapping of mappings) {
  mcp__pixelbyte-figma-mcp__figma_add_code_connect_map({
    params: {
      file_key: "ABC123xyz",
      ...mapping
    }
  });
}
```

**Doğrulama:**

```javascript
// Kayıtları doğrula
const result = mcp__pixelbyte-figma-mcp__figma_get_code_connect_map({
  params: { file_key: "ABC123xyz" }
});

console.log(`Toplam ${result.mappings.length} eşleştirme kaydedildi.`);
```

---

## Code Generation with Code Connect

Kod üretimi sırasında Code Connect nasıl kullanılır.

### Karar Akışı

```
                    ┌─────────────────────┐
                    │   Figma Node ID     │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │ Code Connect Map    │
                    │ Kontrol Et          │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
       ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
       │ Mapping Var │  │ Kısmi Map   │  │ Mapping Yok │
       │ (Exact)     │  │ (Parent)    │  │             │
       └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
              │                │                │
       ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
       │ Import      │  │ Import +    │  │ Generate    │
       │ Existing    │  │ Customize   │  │ New Code    │
       └─────────────┘  └─────────────┘  └─────────────┘
```

### Implementation

```typescript
async function generateCodeWithCodeConnect(
  fileKey: string,
  nodeId: string
): Promise<string> {

  // 1. Önce Code Connect map kontrol et
  const codeConnectResult = await mcp__pixelbyte-figma-mcp__figma_get_code_connect_map({
    params: {
      file_key: fileKey,
      node_id: nodeId
    }
  });

  // 2. Mapping varsa mevcut component'i kullan
  if (codeConnectResult.mappings.length > 0) {
    const mapping = codeConnectResult.mappings[0];

    return generateImportStatement(mapping);
  }

  // 3. Mapping yoksa yeni kod üret
  const generatedCode = await mcp__pixelbyte-figma-mcp__figma_generate_code({
    params: {
      file_key: fileKey,
      node_id: nodeId,
      framework: "react_tailwind"
    }
  });

  return generatedCode;
}

function generateImportStatement(mapping: CodeConnectMapping): string {
  const { component_path, component_name, example, variants, props_mapping } = mapping;

  // Import path oluştur
  const importPath = component_path
    .replace('src/', '@/')
    .replace('.tsx', '');

  return `
// Code Connect: Mevcut component kullanılıyor
import { ${component_name} } from '${importPath}';

// Örnek kullanım:
${example}

// Variant mapping:
// ${JSON.stringify(variants, null, 2)}
`;
}
```

### Kullanım Örneği

```javascript
// Phase 3: Code Generation sırasında

// 1. Node ID al
const nodeId = "123:456";
const fileKey = "ABC123xyz";

// 2. Code Connect kontrol et
const mapping = await mcp__pixelbyte-figma-mcp__figma_get_code_connect_map({
  params: { file_key: fileKey, node_id: nodeId }
});

if (mapping.mappings.length > 0) {
  // ✅ Mapping var - mevcut component kullan
  const { component_path, component_name, example } = mapping.mappings[0];

  console.log(`✅ Mevcut component bulundu: ${component_name}`);
  console.log(`   Path: ${component_path}`);
  console.log(`   Örnek: ${example}`);

  // Import statement üret
  const code = `import { ${component_name} } from '@/${component_path.replace('src/', '').replace('.tsx', '')}';`;

} else {
  // ❌ Mapping yok - yeni kod üret
  console.log(`⚠️ Mapping bulunamadı, yeni kod üretiliyor...`);

  const generatedCode = await mcp__pixelbyte-figma-mcp__figma_generate_code({
    params: {
      file_key: fileKey,
      node_id: nodeId,
      framework: "react_tailwind"
    }
  });

  // Yeni component için mapping öner
  console.log(`💡 Yeni component için Code Connect eklemek ister misiniz?`);
}
```

### Sonuç Karşılaştırması

| Durum | Code Connect | Sonuç |
|-------|--------------|-------|
| Mapping var | ✅ | `import { Button } from '@/components/ui/button'` |
| Mapping yok | ❌ | Yeni component kodu üretilir |
| Kısmi mapping | ⚠️ | Import + custom props/styling |

---

## Best Practices

### 1. Atomik Component'lerden Başla

```
Öncelik Sırası:
1. Button, Input, Badge, Avatar (Atomik)
2. Card, Dialog, Toast (Moleküler)
3. Form, Table, Sidebar (Organizma)
4. Page layouts (Template)
```

**Neden:**
- Atomik component'ler en çok yeniden kullanılır
- Daha az variant = Daha kolay mapping
- Kompozit component'ler atomik olanları kullanır

### 2. Tutarlı İsimlendirme

```
✅ Doğru:
Figma: "Button"      → Kod: Button
Figma: "Primary"     → Kod: variant="default"
Figma: "Small"       → Kod: size="sm"

❌ Yanlış:
Figma: "Btn Primary" → Kod: Button (isim uyumsuz)
Figma: "big"         → Kod: size="lg" (case uyumsuz)
```

### 3. Unmapped Component'leri Dokümante Et

```markdown
## Unmapped Components

| Figma Component | Neden Unmapped | Aksiyon |
|-----------------|----------------|---------|
| HeroSection | Codebase'de yok | Phase 3'te üret |
| CustomLoader | Özel animasyon | Manuel implement |
| LegacyCard | Deprecated | Yeni Card kullan |
```

### 4. Düzenli Güncelle

```
Güncelleme Tetikleyicileri:
- Yeni component eklendiğinde
- Component rename edildiğinde
- Props değiştiğinde
- Variant eklendiğinde/kaldırıldığında
```

**Güncelleme Scripti:**

```javascript
// Aylık Code Connect audit
async function auditCodeConnect(fileKey: string) {
  // 1. Mevcut mappingleri al
  const mappings = await getCodeConnectMap(fileKey);

  // 2. Codebase component'lerini tara
  const codebaseComponents = await scanCodebase();

  // 3. Karşılaştır
  for (const mapping of mappings) {
    const exists = codebaseComponents.includes(mapping.component_path);
    if (!exists) {
      console.warn(`⚠️ Orphan mapping: ${mapping.component_name} - dosya bulunamadı`);
    }
  }
}
```

### 5. Mappingleri Doğrula

```javascript
// Mapping doğrulama fonksiyonu
async function validateMapping(mapping: CodeConnectMapping): Promise<boolean> {
  const checks = {
    pathExists: await fileExists(mapping.component_path),
    componentExported: await isExported(mapping.component_path, mapping.component_name),
    propsValid: await validateProps(mapping.props_mapping),
    exampleCompiles: await compileExample(mapping.example)
  };

  const allValid = Object.values(checks).every(v => v);

  if (!allValid) {
    console.error('Mapping validation failed:', checks);
  }

  return allValid;
}
```

---

## Troubleshooting

| Problem | Olası Neden | Çözüm |
|---------|-------------|-------|
| Mapping bulunamıyor | Yanlış node_id | URL'den node-id al, `-` → `:` çevir |
| Component import hatası | Yanlış path | `src/` prefix kontrolü, dosya varlığı kontrolü |
| Props type error | Mapping uyumsuzluğu | props_mapping değerlerini TypeScript types ile eşleştir |
| Variant çalışmıyor | Yanlış değer mapping | Figma variant adı ile variants objesi key'lerini eşitle |
| Duplicate mapping | Aynı node birden fazla | Önce `remove_code_connect_map` sonra tekrar ekle |
| Mapping kayboldu | Cache/session sorunu | Mappingler kalıcı değilse dosya bazlı saklama düşün |
| Props eksik | Kısmi mapping | props_mapping objesine eksik prop'ları ekle |
| Nested component | Parent mapping var, child yok | Her component için ayrı mapping oluştur |

### Debug Adımları

```javascript
// 1. Mapping'in varlığını kontrol et
const checkMapping = await mcp__pixelbyte-figma-mcp__figma_get_code_connect_map({
  params: { file_key: "ABC123xyz", node_id: "123:456" }
});
console.log('Mapping:', JSON.stringify(checkMapping, null, 2));

// 2. Node'un doğru olduğunu doğrula
const nodeDetails = await mcp__pixelbyte-figma-mcp__figma_get_node_details({
  params: { file_key: "ABC123xyz", node_id: "123:456" }
});
console.log('Node name:', nodeDetails.name);

// 3. Component dosyasının varlığını kontrol et
const fs = require('fs');
const componentPath = 'src/components/ui/button.tsx';
console.log('File exists:', fs.existsSync(componentPath));

// 4. Export kontrolü
const fileContent = fs.readFileSync(componentPath, 'utf8');
console.log('Button exported:', fileContent.includes('export') && fileContent.includes('Button'));
```

### Yaygın Hatalar ve Düzeltmeleri

#### Node ID Format Hatası

```javascript
// ❌ Yanlış (URL formatı)
node_id: "123-456"

// ✅ Doğru (API formatı)
node_id: "123:456"
```

#### Path Format Hatası

```javascript
// ❌ Yanlış
component_path: "@/components/ui/button"
component_path: "components/ui/button.tsx"

// ✅ Doğru
component_path: "src/components/ui/button.tsx"
```

#### Variant Key Uyumsuzluğu

```javascript
// Figma'da: "Variant" property, "Primary" value
// ❌ Yanlış
variants: { "primary": { "variant": "default" } }  // lowercase

// ✅ Doğru (Figma'daki gibi)
variants: { "Primary": { "variant": "default" } }  // Figma case'i ile aynı
```

---

## Quick Reference

### Workflow Özeti

```
Phase 1: Inventory     → find, grep ile codebase tara
Phase 2: Analysis      → figma_get_file_structure, figma_get_node_details
Phase 3: Matching      → Component eşleştirme tablosu oluştur
Phase 4: Props Mapping → props_mapping ve variants tanımla
Phase 5: Register      → figma_add_code_connect_map ile kaydet
```

### MCP Tool Özeti

| Tool | Amaç | Zorunlu Params |
|------|------|----------------|
| `figma_get_code_connect_map` | Mappingleri getir | file_key |
| `figma_add_code_connect_map` | Mapping ekle/güncelle | file_key, node_id, component_path, component_name |
| `figma_remove_code_connect_map` | Mapping sil | file_key, node_id |

### Karar Ağacı

```
Figma component tespit edildi
│
├─ Code Connect mapping var mı?
│  ├─ Evet → Mevcut component'i import et
│  └─ Hayır → Codebase'de benzer component var mı?
│             ├─ Evet → Code Connect mapping oluştur
│             └─ Hayır → Yeni component üret
```
