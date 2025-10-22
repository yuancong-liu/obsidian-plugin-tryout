# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Development Commands

### Building & Development
```bash
# Install dependencies
npm install

# Development mode with watch (auto-recompile on changes)
npm run dev

# Production build (with TypeScript type-checking)
npm run build

# Version bump (updates manifest.json, package.json, and versions.json)
npm run version
```

### Linting
```bash
# Install eslint globally if not already installed
npm install -g eslint

# Lint main.ts
eslint main.ts

# Lint all TypeScript files in a directory (e.g., src/)
eslint ./src/
```

### Testing the Plugin
After building, the plugin can be tested by:
1. Ensuring `main.js`, `manifest.json`, and `styles.css` (if present) are in the plugin directory
2. Reloading Obsidian (Ctrl/Cmd + R)
3. Enabling the plugin in **Settings → Community plugins**

## Architecture Overview

### Build System
- **Bundler**: esbuild (configured in `esbuild.config.mjs`)
- **Entry point**: `main.ts` → compiles to `main.js`
- **External dependencies**: Obsidian API, Electron, CodeMirror modules are marked external (not bundled)
- **Target**: ES2018, CommonJS format
- **Development mode**: Enables source maps and watch mode
- **Production mode**: Minification enabled, no source maps

### Plugin Structure
This is a sample plugin demonstrating the standard Obsidian plugin lifecycle:

- **Plugin class** (`MyPlugin extends Plugin`): Main entry point with `onload()` and `onunload()` lifecycle methods
- **Settings**: Interface-based settings with `loadSettings()` / `saveSettings()` persistence via `loadData()` / `saveData()`
- **Commands**: Registered via `addCommand()` with unique IDs
- **UI Components**: Modals (`Modal`), settings tabs (`PluginSettingTab`), ribbon icons, status bar items
- **Event Management**: Uses `registerDomEvent()` and `registerInterval()` for automatic cleanup on plugin disable

### Key Patterns
- **Settings persistence**: Settings are loaded on plugin initialization and saved asynchronously when changed
- **Resource cleanup**: All DOM events and intervals must be registered using `register*` helpers to ensure proper cleanup on unload
- **Command registration**: Commands have stable IDs and can include `callback`, `editorCallback`, or `checkCallback` for conditional execution
- **Mobile compatibility**: `isDesktopOnly: false` in manifest.json means this plugin should avoid desktop-only APIs

## Critical Constraints

### Manifest Requirements
- `id` must never change after initial release (treat as stable API)
- `version` must follow SemVer (x.y.z)
- `minAppVersion` must be accurate when using newer Obsidian APIs
- Folder name during development should match the `id` field

### Build Artifacts
- **Never commit**: `node_modules/`, `main.js`, or generated build artifacts to version control
- **Release artifacts**: `main.js`, `manifest.json`, `styles.css` (if present) must be at plugin root level
- All dependencies must be bundled into `main.js` (except externals listed in esbuild config)

### Code Organization Best Practices
- Keep `main.ts` minimal—focus on plugin lifecycle (onload, onunload, command registration)
- Split functionality into separate modules when files exceed ~200-300 lines
- Suggested structure for larger plugins:
  ```
  src/
    main.ts           # Plugin lifecycle only
    settings.ts       # Settings interface and defaults
    commands/         # Command implementations
    ui/              # Modals, views, components
    utils/           # Helper functions
    types.ts         # TypeScript interfaces
  ```

### Version Control & Releases
- GitHub release tag must exactly match `manifest.json` version (no `v` prefix)
- Update `versions.json` to map plugin version → minimum Obsidian version
- Attach `manifest.json`, `main.js`, and `styles.css` as binary release assets
- Use `npm run version` after manually updating `minAppVersion` to automate version bumps

## TypeScript Configuration
- Strict null checks enabled
- Target: ES6, Module: ESNext
- Inline source maps for development
- `noImplicitAny` enforced for type safety
