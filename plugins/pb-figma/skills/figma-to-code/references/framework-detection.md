# Framework Detection Reference

> **Used by:** code-generator-react, code-generator-swiftui, code-generator-vue, code-generator-kotlin, SKILL.md orchestration

## Standard Detection Flow

1. **Scan** project root for framework indicators (file patterns, package.json, build files)
2. **Confirm** with user via `AskUserQuestion` if detection is ambiguous
3. **Map** to MCP code generation parameter for `figma_generate_code`

## Detection Matrix

| Priority | Indicator Files | Framework | MCP Parameter | Agent |
|----------|----------------|-----------|---------------|-------|
| 1 | `*.xcodeproj`, `*.xcworkspace`, `Package.swift` | SwiftUI | `swiftui` | code-generator-swiftui |
| 2 | `build.gradle.kts` with `androidx.compose` | Kotlin Compose | `kotlin` | code-generator-kotlin 🚧 |
| 3 | `package.json` with `"vue"` or `"nuxt"` | Vue/Nuxt | `vue_tailwind` | code-generator-vue 🚧 |
| 4 | `package.json` with `"react"` or `"next"` | React/Next.js | `react_tailwind` | code-generator-react |
| 5 | None detected | React (default) | `react_tailwind` | code-generator-react |

> **Priority order matters:** Check Swift/Xcode first (most distinctive), then Kotlin, then Vue (before React since both are in package.json), then React as default.

## Variant Detection

### React Variants
| Variant | Detection | Notes |
|---------|-----------|-------|
| Next.js | `next.config.*` exists OR `"next"` in package.json dependencies | App Router vs Pages Router: check for `app/` directory |
| Vite | `vite.config.*` exists | Check for `"@vitejs/plugin-react"` |
| CRA | `"react-scripts"` in package.json | Legacy — suggest migration |

### SwiftUI Variants
| Variant | Detection | Notes |
|---------|-----------|-------|
| Xcode Project | `*.xcodeproj` exists | Standard iOS/macOS app |
| Xcode Workspace | `*.xcworkspace` exists | Multi-target or CocoaPods |
| Swift Package | `Package.swift` without `.xcodeproj` | Library or modular app |

### Target Platform Detection (SwiftUI)
- `TARGETED_DEVICE_FAMILY = 1` → iPhone only
- `TARGETED_DEVICE_FAMILY = 2` → iPad only
- `TARGETED_DEVICE_FAMILY = "1,2"` → Universal
- Check `Info.plist` or `.pbxproj` for deployment target

### Tailwind Detection (Web Frameworks)
| Signal | Confidence |
|--------|-----------|
| `tailwind.config.*` exists | High |
| `"tailwindcss"` in devDependencies | High |
| `@tailwind` directives in CSS files | Medium |
| None found | Warn user, suggest adding Tailwind |

## User Confirmation

When detection is ambiguous (multiple frameworks detected):

```
AskUserQuestion:
  question: "Multiple frameworks detected. Which should I use for code generation?"
  options:
    - label: "{detected_framework_1}"
      description: "Found: {indicator_files}"
    - label: "{detected_framework_2}"
      description: "Found: {indicator_files}"
```

When no framework detected:
```
AskUserQuestion:
  question: "No framework detected in project root. Which framework should I generate code for?"
  options:
    - label: "React + Tailwind (Recommended)"
      description: "Default web framework with utility-first CSS"
    - label: "SwiftUI"
      description: "Native iOS/macOS framework"
```

## MCP Parameter Mapping

| Framework | `figma_generate_code` parameter |
|-----------|-------------------------------|
| React + Tailwind | `framework: "react_tailwind"` |
| React (no Tailwind) | `framework: "react"` |
| SwiftUI | `framework: "swiftui"` |
| Vue + Tailwind | `framework: "vue_tailwind"` |
| Vue (no Tailwind) | `framework: "vue"` |
| Kotlin Compose | `framework: "kotlin"` |
| HTML + CSS | `framework: "html_css"` |
