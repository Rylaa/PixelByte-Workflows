# Font Manager Agent Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Figma tasarımından fontları tespit eden, çoklu kaynaklardan indiren ve platforma uygun şekilde projeye kuran background agent oluşturmak.

**Architecture:** Design Validator tamamlandığında background'da tetiklenen agent, Figma'dan typography token'larını çeker, Google Fonts/Adobe Fonts/Font Squirrel'dan font dosyalarını indirir, projenin framework'ünü tespit ederek (React, SwiftUI, Kotlin, Vue) platforma uygun kurulum yapar ve sonuçları spec dosyasına yazar.

**Tech Stack:** Claude Code Agent, Figma MCP, Google Fonts API, WebFetch, Bash

---

## Task 1: Agent Dosyası Oluştur

**Files:**
- Create: `plugins/pb-figma/agents/font-manager.md`

**Step 1: Agent frontmatter ve temel yapıyı yaz**

```markdown
---
name: font-manager
description: >
  Figma tasarımından fontları tespit eder, çoklu kaynaklardan (Google Fonts,
  Adobe Fonts, Font Squirrel) indirir ve platforma uygun şekilde projeye kurar.
  Design Validator sonrası background'da çalışır, pipeline'ı bloklamaz.
tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - WebFetch
  - mcp__plugin_pb-figma_pixelbyte-figma-mcp__figma_get_design_tokens
  - mcp__plugin_pb-figma_pixelbyte-figma-mcp__figma_get_styles
  - TodoWrite
  - AskUserQuestion
---

# Font Manager Agent

You manage fonts for Figma-to-code projects. You detect required fonts from Figma designs, download them from multiple sources, and set them up according to the target platform.

## Trigger

This agent runs as a **background process** after Design Validator completes successfully. It does not block the main pipeline.

**Trigger condition:** Design Validator outputs status PASS or WARN (not FAIL)

## Input

Read the Validation Report from: `docs/figma-reports/{file_key}-validation.md`

### Extracting Font Information

From the Validation Report, extract fonts from the **Typography** section:

| Field | Source |
|-------|--------|
| Font Family | Typography table "Font" column |
| Font Weights | Typography table "Weight" column |
| Font Styles | Infer from usage (regular, italic) |

**Example extraction:**
```
From:
| Style | Font | Size | Weight | Line Height |
|-------|------|------|--------|-------------|
| heading-1 | Inter | 32px | 700 | 1.2 |
| body | Inter | 16px | 400 | 1.5 |
| caption | Roboto | 12px | 500 | 1.4 |

Extract:
- Inter: weights [400, 700]
- Roboto: weights [500]
```
```

**Step 2: Dosyayı kaydet**

Run: `ls -la plugins/pb-figma/agents/font-manager.md`
Expected: File exists with correct permissions

**Step 3: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add font-manager agent skeleton"
```

---

## Task 2: Font Tespit Süreci

**Files:**
- Modify: `plugins/pb-figma/agents/font-manager.md`

**Step 1: Font tespit sürecini ekle**

Aşağıdaki bölümü agent dosyasına ekle (Input bölümünden sonra):

```markdown
## Process

Use `TodoWrite` to track font management progress:

1. **Read Validation Report** - Parse typography section
2. **Extract Unique Fonts** - Build font family + weights list
3. **Detect Project Platform** - Identify React/Swift/Kotlin/Vue
4. **Check Local Availability** - See if fonts already exist in project
5. **Search Font Sources** - Query Google Fonts, Adobe, Font Squirrel
6. **Download Fonts** - Fetch font files from best source
7. **Setup for Platform** - Configure fonts per platform requirements
8. **Update Spec** - Add "Fonts Setup" section to spec file

## Font Detection

### Step 1: Parse Typography from Validation Report

```bash
# Read the validation report
Read("docs/figma-reports/{file_key}-validation.md")
```

Extract from the Typography table:
- Font family names (e.g., "Inter", "Roboto", "SF Pro")
- Font weights used (e.g., 400, 500, 700)
- Infer styles (regular, italic based on naming)

### Step 2: Direct Figma Verification (Optional)

If validation report lacks detail, fetch directly:

```
figma_get_design_tokens:
  - file_key: {file_key}
  - include_typography: true
```

This returns comprehensive typography tokens including:
- fontFamily
- fontWeight
- fontSize
- lineHeight
- letterSpacing

### Step 3: Build Font Requirements List

Create a structured list:

```
fonts_required:
  - family: "Inter"
    weights: [400, 500, 600, 700]
    styles: [normal]
    source: null  # to be determined

  - family: "Roboto"
    weights: [400, 700]
    styles: [normal, italic]
    source: null
```
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add font detection process to font-manager"
```

---

## Task 3: Platform Tespit Mantığı

**Files:**
- Modify: `plugins/pb-figma/agents/font-manager.md`

**Step 1: Platform detection bölümünü ekle**

```markdown
## Platform Detection

Detect the target platform by checking project files:

### Detection Rules

| Check | Platform | Setup Method |
|-------|----------|--------------|
| `package.json` has "next" | Next.js | `next/font` or `public/fonts` |
| `package.json` has "react" (no next) | React | `public/fonts` + CSS |
| `package.json` has "vue" | Vue | `public/fonts` + CSS |
| `Podfile` or `*.xcodeproj` exists | SwiftUI/iOS | Bundle + Info.plist |
| `build.gradle` or `build.gradle.kts` | Kotlin/Android | `res/font` + XML |

### Detection Commands

```bash
# Check for Next.js
Grep("\"next\"", "package.json")

# Check for React (vanilla)
Grep("\"react\"", "package.json") && ! Grep("\"next\"", "package.json")

# Check for Vue
Grep("\"vue\"", "package.json")

# Check for iOS/SwiftUI
Glob("**/*.xcodeproj") || Glob("**/Podfile")

# Check for Android/Kotlin
Glob("**/build.gradle") || Glob("**/build.gradle.kts")
```

### Platform Priority

If multiple platforms detected (monorepo), ask user:

```
AskUserQuestion:
  question: "Multiple platforms detected. Which one should I set up fonts for?"
  options:
    - "Next.js/React"
    - "SwiftUI/iOS"
    - "Kotlin/Android"
    - "Vue"
    - "All platforms"
```
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add platform detection to font-manager"
```

---

## Task 4: Font Kaynak Arama Sistemi

**Files:**
- Modify: `plugins/pb-figma/agents/font-manager.md`

**Step 1: Font source search bölümünü ekle**

```markdown
## Font Source Search

Search for fonts in this priority order:

### 1. Google Fonts (Primary)

```
WebFetch:
  url: "https://fonts.google.com/download?family={font_family}"
  prompt: "Download the font family zip file"
```

Alternative API check:
```
WebFetch:
  url: "https://fonts.googleapis.com/css2?family={font_family}:wght@{weights}"
  prompt: "Get the font CSS with download URLs"
```

**Google Fonts URL Pattern:**
- CSS: `https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap`
- Download: `https://fonts.google.com/download?family=Inter`

### 2. Font Squirrel (Fallback 1)

```
WebFetch:
  url: "https://www.fontsquirrel.com/fonts/{font-family-slug}"
  prompt: "Check if font exists and get download link"
```

**Note:** Font Squirrel uses kebab-case slugs (e.g., "open-sans")

### 3. Adobe Fonts Check (Fallback 2)

```
WebFetch:
  url: "https://fonts.adobe.com/fonts/{font-family-slug}"
  prompt: "Check if font exists on Adobe Fonts"
```

**Note:** Adobe Fonts requires subscription. If found here, report to user but don't auto-download.

### Search Algorithm

```
For each font_family in fonts_required:
  1. Try Google Fonts API
     - If found: mark source = "google", get download URL
     - If not found: continue

  2. Try Font Squirrel
     - If found: mark source = "fontsquirrel", get download URL
     - If not found: continue

  3. Check Adobe Fonts
     - If found: mark source = "adobe", note "requires subscription"
     - If not found: continue

  4. If no source found:
     - Mark as "not_found"
     - Prepare fallback suggestion
```

### Fallback Font Mapping

When a font cannot be found, suggest alternatives:

| Original Font | Fallback Options |
|---------------|------------------|
| SF Pro | Inter, -apple-system, system-ui |
| SF Pro Display | Inter, -apple-system |
| SF Pro Text | Inter, -apple-system |
| Helvetica Neue | Inter, Arial, sans-serif |
| Roboto | Inter, -apple-system, sans-serif |
| Open Sans | Inter, Source Sans Pro, sans-serif |
| Montserrat | Poppins, Inter, sans-serif |
| Playfair Display | Merriweather, Georgia, serif |
| Custom/Unknown | Inter, system-ui, sans-serif |

When fallback is needed:
```
AskUserQuestion:
  question: "Font '{original}' not found in free sources. What should I do?"
  options:
    - "Use fallback: {fallback_suggestion}"
    - "Skip this font (use system default)"
    - "I'll provide the font file manually"
```
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add font source search system to font-manager"
```

---

## Task 5: Font İndirme ve Kurulum - React/Next.js

**Files:**
- Modify: `plugins/pb-figma/agents/font-manager.md`

**Step 1: React/Next.js font setup bölümünü ekle**

```markdown
## Platform Setup: React/Next.js

### Directory Structure

```
project/
├── public/
│   └── fonts/
│       ├── Inter-Regular.woff2
│       ├── Inter-Medium.woff2
│       ├── Inter-SemiBold.woff2
│       └── Inter-Bold.woff2
├── src/
│   └── styles/
│       └── fonts.css
└── package.json
```

### Step 1: Create Fonts Directory

```bash
mkdir -p public/fonts
```

### Step 2: Download Font Files

For Google Fonts, download and extract:

```bash
# Download font family
curl -L "https://fonts.google.com/download?family={FontFamily}" -o /tmp/{font-family}.zip

# Extract to public/fonts
unzip -o /tmp/{font-family}.zip -d /tmp/{font-family}

# Copy woff2 files (preferred format)
cp /tmp/{font-family}/static/*.woff2 public/fonts/ 2>/dev/null || \
cp /tmp/{font-family}/*.ttf public/fonts/
```

### Step 3: Create CSS File

Write to `src/styles/fonts.css`:

```css
/* Inter Font Family */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-Regular.woff2') format('woff2'),
       url('/fonts/Inter-Regular.ttf') format('truetype');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-Medium.woff2') format('woff2'),
       url('/fonts/Inter-Medium.ttf') format('truetype');
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-SemiBold.woff2') format('woff2'),
       url('/fonts/Inter-SemiBold.ttf') format('truetype');
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-Bold.woff2') format('woff2'),
       url('/fonts/Inter-Bold.ttf') format('truetype');
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}
```

### Step 4: Import in Global Styles

Add to `src/styles/globals.css` or `src/app/globals.css`:

```css
@import './fonts.css';

:root {
  --font-primary: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
}

body {
  font-family: var(--font-primary);
}
```

### Next.js Optimization (Alternative)

For Next.js projects, recommend using `next/font`:

```typescript
// src/app/layout.tsx or pages/_app.tsx
import { Inter } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  weight: ['400', '500', '600', '700'],
  variable: '--font-inter',
  display: 'swap',
});

export default function RootLayout({ children }) {
  return (
    <html className={inter.variable}>
      <body>{children}</body>
    </html>
  );
}
```

**Note:** `next/font` provides automatic optimization but requires code changes. Offer both options to user.
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add React/Next.js font setup to font-manager"
```

---

## Task 6: Font Kurulum - SwiftUI/iOS

**Files:**
- Modify: `plugins/pb-figma/agents/font-manager.md`

**Step 1: SwiftUI font setup bölümünü ekle**

```markdown
## Platform Setup: SwiftUI/iOS

### Directory Structure

```
project/
├── {ProjectName}/
│   ├── Resources/
│   │   └── Fonts/
│   │       ├── Inter-Regular.ttf
│   │       ├── Inter-Medium.ttf
│   │       ├── Inter-SemiBold.ttf
│   │       └── Inter-Bold.ttf
│   ├── Info.plist
│   └── {ProjectName}App.swift
└── {ProjectName}.xcodeproj
```

### Step 1: Find Project Structure

```bash
# Find xcodeproj
Glob("**/*.xcodeproj")

# Find existing Resources folder or Sources
Glob("**/Resources") || Glob("**/Sources")
```

### Step 2: Create Fonts Directory

```bash
# Determine project root (parent of .xcodeproj)
PROJECT_ROOT=$(dirname $(find . -name "*.xcodeproj" -type d | head -1))

# Create fonts directory
mkdir -p "$PROJECT_ROOT/Resources/Fonts"
```

### Step 3: Download and Copy Fonts

```bash
# Download font
curl -L "https://fonts.google.com/download?family={FontFamily}" -o /tmp/{font-family}.zip
unzip -o /tmp/{font-family}.zip -d /tmp/{font-family}

# Copy TTF files (iOS prefers TTF/OTF)
cp /tmp/{font-family}/static/*.ttf "$PROJECT_ROOT/Resources/Fonts/" 2>/dev/null || \
cp /tmp/{font-family}/*.ttf "$PROJECT_ROOT/Resources/Fonts/"
```

### Step 4: Update Info.plist

Add fonts to `Info.plist`:

```xml
<key>UIAppFonts</key>
<array>
    <string>Fonts/Inter-Regular.ttf</string>
    <string>Fonts/Inter-Medium.ttf</string>
    <string>Fonts/Inter-SemiBold.ttf</string>
    <string>Fonts/Inter-Bold.ttf</string>
</array>
```

**Implementation:**

```bash
# Check if UIAppFonts key exists
grep -q "UIAppFonts" "$PROJECT_ROOT/Info.plist"

# If not, need to add it before closing </dict></plist>
```

Or use PlistBuddy:
```bash
/usr/libexec/PlistBuddy -c "Add :UIAppFonts array" "$PROJECT_ROOT/Info.plist" 2>/dev/null
/usr/libexec/PlistBuddy -c "Add :UIAppFonts:0 string 'Fonts/Inter-Regular.ttf'" "$PROJECT_ROOT/Info.plist"
/usr/libexec/PlistBuddy -c "Add :UIAppFonts:1 string 'Fonts/Inter-Medium.ttf'" "$PROJECT_ROOT/Info.plist"
# ... repeat for each font
```

### Step 5: Create Font Extension (Optional)

Create `FontExtensions.swift`:

```swift
import SwiftUI

extension Font {
    static func inter(_ size: CGFloat, weight: Font.Weight = .regular) -> Font {
        let fontName: String
        switch weight {
        case .regular:
            fontName = "Inter-Regular"
        case .medium:
            fontName = "Inter-Medium"
        case .semibold:
            fontName = "Inter-SemiBold"
        case .bold:
            fontName = "Inter-Bold"
        default:
            fontName = "Inter-Regular"
        }
        return .custom(fontName, size: size)
    }
}

// Usage:
// Text("Hello").font(.inter(16, weight: .medium))
```

### Important Notes

1. **Xcode Project Update Required:** Fonts must be added to Xcode project manually or via script. The agent documents this requirement.

2. **Font Names:** iOS uses the PostScript name, not filename. Check with:
   ```bash
   fc-scan --format "%{postscriptname}\n" Inter-Regular.ttf
   ```

3. **Bundle Target:** Ensure fonts are included in the app target's "Copy Bundle Resources" build phase.
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add SwiftUI/iOS font setup to font-manager"
```

---

## Task 7: Font Kurulum - Kotlin/Android

**Files:**
- Modify: `plugins/pb-figma/agents/font-manager.md`

**Step 1: Kotlin/Android font setup bölümünü ekle**

```markdown
## Platform Setup: Kotlin/Android

### Directory Structure

```
project/
├── app/
│   └── src/
│       └── main/
│           ├── res/
│           │   └── font/
│           │       ├── inter_regular.ttf
│           │       ├── inter_medium.ttf
│           │       ├── inter_semibold.ttf
│           │       ├── inter_bold.ttf
│           │       └── inter.xml
│           └── java/...
└── build.gradle
```

### Step 1: Find Android Project Structure

```bash
# Find app module
Glob("**/app/src/main")

# Find existing res folder
Glob("**/res")
```

### Step 2: Create Font Directory

```bash
# Find the res directory
RES_DIR=$(find . -path "*/app/src/main/res" -type d | head -1)

# Create font directory
mkdir -p "$RES_DIR/font"
```

### Step 3: Download and Copy Fonts

```bash
# Download font
curl -L "https://fonts.google.com/download?family={FontFamily}" -o /tmp/{font-family}.zip
unzip -o /tmp/{font-family}.zip -d /tmp/{font-family}

# Copy TTF files with Android naming (lowercase, underscores)
cp /tmp/{font-family}/static/Inter-Regular.ttf "$RES_DIR/font/inter_regular.ttf"
cp /tmp/{font-family}/static/Inter-Medium.ttf "$RES_DIR/font/inter_medium.ttf"
cp /tmp/{font-family}/static/Inter-SemiBold.ttf "$RES_DIR/font/inter_semibold.ttf"
cp /tmp/{font-family}/static/Inter-Bold.ttf "$RES_DIR/font/inter_bold.ttf"
```

**Important:** Android resource names must be lowercase with underscores only.

### Step 4: Create Font Family XML

Write to `res/font/inter.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<font-family xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <font
        android:font="@font/inter_regular"
        android:fontStyle="normal"
        android:fontWeight="400"
        app:font="@font/inter_regular"
        app:fontStyle="normal"
        app:fontWeight="400" />

    <font
        android:font="@font/inter_medium"
        android:fontStyle="normal"
        android:fontWeight="500"
        app:font="@font/inter_medium"
        app:fontStyle="normal"
        app:fontWeight="500" />

    <font
        android:font="@font/inter_semibold"
        android:fontStyle="normal"
        android:fontWeight="600"
        app:font="@font/inter_semibold"
        app:fontStyle="normal"
        app:fontWeight="600" />

    <font
        android:font="@font/inter_bold"
        android:fontStyle="normal"
        android:fontWeight="700"
        app:font="@font/inter_bold"
        app:fontStyle="normal"
        app:fontWeight="700" />

</font-family>
```

### Step 5: Create Compose Typography (Optional)

For Jetpack Compose projects, create `Type.kt`:

```kotlin
package com.example.app.ui.theme

import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.Font
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.sp
import com.example.app.R

val InterFontFamily = FontFamily(
    Font(R.font.inter_regular, FontWeight.Normal),
    Font(R.font.inter_medium, FontWeight.Medium),
    Font(R.font.inter_semibold, FontWeight.SemiBold),
    Font(R.font.inter_bold, FontWeight.Bold)
)

val Typography = Typography(
    bodyLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp
    ),
    titleLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 22.sp,
        lineHeight = 28.sp
    ),
    // ... other styles
)
```

### Downloadable Fonts Alternative

For Google Fonts, Android supports downloadable fonts:

```xml
<!-- res/font/inter.xml -->
<?xml version="1.0" encoding="utf-8"?>
<font-family xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:fontProviderAuthority="com.google.android.gms.fonts"
    android:fontProviderPackage="com.google.android.gms"
    android:fontProviderQuery="Inter"
    android:fontProviderCerts="@array/com_google_android_gms_fonts_certs"
    app:fontProviderAuthority="com.google.android.gms.fonts"
    app:fontProviderPackage="com.google.android.gms"
    app:fontProviderQuery="Inter"
    app:fontProviderCerts="@array/com_google_android_gms_fonts_certs">
</font-family>
```

**Note:** Downloadable fonts require Google Play Services and network at first load.
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add Kotlin/Android font setup to font-manager"
```

---

## Task 8: Font Kurulum - Vue

**Files:**
- Modify: `plugins/pb-figma/agents/font-manager.md`

**Step 1: Vue font setup bölümünü ekle**

```markdown
## Platform Setup: Vue

### Directory Structure

```
project/
├── public/
│   └── fonts/
│       ├── Inter-Regular.woff2
│       ├── Inter-Medium.woff2
│       └── ...
├── src/
│   ├── assets/
│   │   └── styles/
│   │       └── fonts.css
│   ├── App.vue
│   └── main.ts
└── package.json
```

### Step 1: Create Fonts Directory

```bash
mkdir -p public/fonts
```

### Step 2: Download Font Files

Same as React:
```bash
curl -L "https://fonts.google.com/download?family={FontFamily}" -o /tmp/{font-family}.zip
unzip -o /tmp/{font-family}.zip -d /tmp/{font-family}
cp /tmp/{font-family}/static/*.woff2 public/fonts/ 2>/dev/null || \
cp /tmp/{font-family}/*.ttf public/fonts/
```

### Step 3: Create CSS File

Write to `src/assets/styles/fonts.css`:

```css
@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-Regular.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-Medium.woff2') format('woff2');
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}

/* ... more weights */
```

### Step 4: Import in Main Entry

Add to `src/main.ts` or `src/main.js`:

```typescript
import './assets/styles/fonts.css'
```

Or in `src/App.vue`:

```vue
<style>
@import './assets/styles/fonts.css';

:root {
  --font-primary: 'Inter', -apple-system, sans-serif;
}
</style>
```
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add Vue font setup to font-manager"
```

---

## Task 9: Spec Güncelleme ve Çıktı Formatı

**Files:**
- Modify: `plugins/pb-figma/agents/font-manager.md`

**Step 1: Output format bölümünü ekle**

```markdown
## Output: Update Spec File

Modify the Implementation Spec at: `docs/figma-reports/{file_key}-spec.md`

### Add "Fonts Setup" Section

Insert after "Design Tokens" section:

```markdown
## Fonts Setup

**Status:** COMPLETE | PARTIAL | FAILED
**Platform:** {detected_platform}
**Generated:** {timestamp}

### Fonts Required

| Font Family | Weights | Source | Status |
|-------------|---------|--------|--------|
| Inter | 400, 500, 600, 700 | Google Fonts | ✅ Downloaded |
| Roboto | 400, 700 | Google Fonts | ✅ Downloaded |
| SF Pro | 400, 600 | Not Found | ⚠️ Using fallback: Inter |

### Files Created

| File | Purpose |
|------|---------|
| `public/fonts/Inter-Regular.woff2` | Inter 400 weight |
| `public/fonts/Inter-Medium.woff2` | Inter 500 weight |
| `public/fonts/Inter-SemiBold.woff2` | Inter 600 weight |
| `public/fonts/Inter-Bold.woff2` | Inter 700 weight |
| `src/styles/fonts.css` | @font-face definitions |

### Configuration Added

#### For React/Next.js:
```css
/* Added to src/styles/globals.css */
@import './fonts.css';

:root {
  --font-primary: 'Inter', -apple-system, sans-serif;
}
```

#### For SwiftUI:
```xml
<!-- Added to Info.plist -->
<key>UIAppFonts</key>
<array>
  <string>Fonts/Inter-Regular.ttf</string>
  ...
</array>
```

#### For Kotlin/Android:
```
Created: res/font/inter.xml
Created: res/font/inter_regular.ttf, inter_medium.ttf, ...
```

### Usage Examples

#### React/Next.js
```tsx
<h1 className="font-['Inter'] font-semibold">Title</h1>
// or with CSS variable
<h1 style={{ fontFamily: 'var(--font-primary)' }}>Title</h1>
```

#### SwiftUI
```swift
Text("Title")
    .font(.custom("Inter-SemiBold", size: 24))
// or with extension
Text("Title")
    .font(.inter(24, weight: .semibold))
```

#### Kotlin/Android
```kotlin
Text(
    text = "Title",
    fontFamily = InterFontFamily,
    fontWeight = FontWeight.SemiBold
)
```

### Warnings

- {any warnings or notes}

### Manual Steps Required

- [ ] {any manual steps the user needs to complete}
```
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add output format to font-manager"
```

---

## Task 10: Hata Yönetimi ve Fallback Sistemi

**Files:**
- Modify: `plugins/pb-figma/agents/font-manager.md`

**Step 1: Error handling bölümünü ekle**

```markdown
## Error Handling

### Retry Logic

```
MAX_RETRIES = 3
Retry on: timeout, network_error, rate_limit
Backoff: 2s, 4s, 8s
```

### Error Matrix

| Error | Recovery | Action |
|-------|----------|--------|
| Font not found | Suggest fallback | AskUserQuestion with alternatives |
| Download failed | Retry 3x | If fails, skip and document |
| Invalid font file | Re-download | If fails, suggest alternative |
| Platform not detected | Ask user | AskUserQuestion for platform |
| Spec not found | Stop | Report error, wait for pipeline |
| Permission denied | Document | Note in output, provide manual steps |

### Fallback Decision Flow

```
Font "{name}" not found in any source:
  1. Check fallback mapping table
  2. If fallback exists:
     - AskUserQuestion: "Use {fallback} instead of {original}?"
     - If yes: Download and use fallback
     - If no: Skip font, document as missing
  3. If no fallback:
     - Document as "Not available"
     - Suggest system font stack
```

### Partial Success Handling

Continue processing if:
- At least 50% of fonts downloaded
- Primary/heading fonts available

Stop and report if:
- No fonts could be downloaded
- All downloads failed

### Rate Limit Handling

Google Fonts API:
- No official rate limit, but be respectful
- Add 1s delay between downloads

Font Squirrel:
- If blocked, wait 30s and retry once
- Document if still blocked

### Validation Checklist

Before completing, verify:

- [ ] All available fonts downloaded
- [ ] Font files exist in correct directories
- [ ] Configuration files created/updated
- [ ] Spec file updated with "Fonts Setup" section
- [ ] Warnings documented for missing fonts
- [ ] Manual steps clearly listed
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/font-manager.md
git commit -m "feat(pb-figma): add error handling to font-manager"
```

---

## Task 11: README Güncelleme

**Files:**
- Modify: `plugins/pb-figma/agents/README.md`

**Step 1: README'e font-manager'ı ekle**

Mevcut README'e ekle:

```markdown
## Background Agents

### font-manager

Runs in background after Design Validator completes. Does not block the main pipeline.

**Trigger:** Design Validator status is PASS or WARN

**Function:**
- Detects fonts from Figma typography tokens
- Downloads from Google Fonts, Font Squirrel
- Sets up fonts for detected platform (React, SwiftUI, Kotlin, Vue)
- Updates spec with "Fonts Setup" section

**Usage:**
```
Pipeline runs automatically:
Design Validator ──┬──► Design Analyst ──► ...
                   │
                   └──► Font Manager (background)
```

Manual trigger:
```
@font-manager setup docs/figma-reports/{file_key}-validation.md
```
```

**Step 2: Commit**

```bash
git add plugins/pb-figma/agents/README.md
git commit -m "docs(pb-figma): add font-manager to README"
```

---

## Task 12: Final Test ve Doğrulama

**Files:**
- Read: `plugins/pb-figma/agents/font-manager.md`

**Step 1: Agent dosyasının tamamını doğrula**

```bash
# Dosya var mı kontrol et
ls -la plugins/pb-figma/agents/font-manager.md

# Frontmatter doğru mu
head -20 plugins/pb-figma/agents/font-manager.md

# Tüm bölümler var mı
grep -E "^## " plugins/pb-figma/agents/font-manager.md
```

Expected output for headers:
```
## Trigger
## Input
## Process
## Font Detection
## Platform Detection
## Font Source Search
## Platform Setup: React/Next.js
## Platform Setup: SwiftUI/iOS
## Platform Setup: Kotlin/Android
## Platform Setup: Vue
## Output: Update Spec File
## Error Handling
```

**Step 2: Syntax kontrolü**

```bash
# YAML frontmatter geçerli mi
head -15 plugins/pb-figma/agents/font-manager.md | grep -E "^(name|description|tools):"
```

**Step 3: Final commit**

```bash
git add -A
git commit -m "feat(pb-figma): complete font-manager agent implementation

- Add font detection from Figma typography tokens
- Support multiple font sources (Google Fonts, Font Squirrel)
- Platform-specific setup for React, SwiftUI, Kotlin, Vue
- Fallback font suggestions when fonts not found
- Spec file integration with Fonts Setup section
- Comprehensive error handling and retry logic"
```

---

## Summary

| Task | Description | Files |
|------|-------------|-------|
| 1 | Agent skeleton | `agents/font-manager.md` |
| 2 | Font detection | `agents/font-manager.md` |
| 3 | Platform detection | `agents/font-manager.md` |
| 4 | Font source search | `agents/font-manager.md` |
| 5 | React/Next.js setup | `agents/font-manager.md` |
| 6 | SwiftUI/iOS setup | `agents/font-manager.md` |
| 7 | Kotlin/Android setup | `agents/font-manager.md` |
| 8 | Vue setup | `agents/font-manager.md` |
| 9 | Output format | `agents/font-manager.md` |
| 10 | Error handling | `agents/font-manager.md` |
| 11 | README update | `agents/README.md` |
| 12 | Final verification | - |

**Total estimated tasks:** 12
**Primary file:** `plugins/pb-figma/agents/font-manager.md`
