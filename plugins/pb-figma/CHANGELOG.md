# Changelog

All notable changes to the pb-figma plugin will be documented in this file.

## [1.4.0] - 2026-01-26

### Added
- **Text Decoration Support** - Design Analyst now extracts underline/strikethrough colors
- **COMPLEX_VECTOR Classification** - Asset Manager detects multi-path vectors (≥10 paths, >100px) and downloads as PNG
- **SwiftUI Text Decoration** - Code Generator applies `.underline(color:)` and `.strikethrough(color:)` modifiers
- **E2E Test Suite** - Complete test coverage for TypeSceneText and OnboardingAnalysisView components

### Fixed
- **Opacity Extraction** - Design Analyst now correctly calculates compound opacity (fill opacity × node opacity)
- **Gradient Detection** - All gradient stops preserved with exact positions (was only keeping first/last)
- **SwiftUI Opacity Application** - Code Generator now applies `.opacity()` modifiers from spec
- **SwiftUI Gradient Rendering** - AngularGradient/LinearGradient now include all color stops with exact locations

### Changed
- Asset Manager classification table updated with clear criteria for SIMPLE_ICON vs COMPLEX_VECTOR vs RASTER_IMAGE
- Design Analyst opacity column now shows "Usage" context (fill, stroke, overlay, etc.)

## [1.3.0] - 2026-01-25

### Added
- Initial 5-agent pipeline architecture
- Design Validator agent
- Design Analyst agent
- Asset Manager agent
- Code Generator agents (React, SwiftUI, Vue, Kotlin)
- Compliance Checker agent
- figma-to-code skill with framework detection

### Features
- Pixelbyte Figma MCP Server integration
- Auto Layout validation
- Design token extraction
- Code Connect support
- Visual validation with Claude Vision
