# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chrome Extension boilerplate using React 19, TypeScript 5.9, Vite 6.3, and Turborepo. Supports both Chrome and Firefox with Manifest V3. This is a pnpm monorepo with 24 workspace packages.

**Requirements**: Node.js >= 22.15.1, pnpm 10.11.0

## Common Commands

### Development
```bash
pnpm dev              # Start dev server with HMR (Chrome)
pnpm dev:firefox      # Start dev server with HMR (Firefox)
```

After running dev, load the extension in browser:
- **Chrome**: `chrome://extensions` → Enable "Developer mode" → "Load unpacked" → select `dist/` directory
- **Firefox**: `about:debugging#/runtime/this-firefox` → "Load Temporary Add-on" → select `dist/manifest.json`

### Build & Package
```bash
pnpm build            # Production build (Chrome)
pnpm build:firefox    # Production build (Firefox)
pnpm zip              # Build and create zip in dist-zip/
```

### Code Quality
```bash
pnpm lint             # Run ESLint on all packages
pnpm lint:fix         # Auto-fix ESLint issues
pnpm format           # Run Prettier on all packages
pnpm type-check       # TypeScript type checking
```

### Testing
```bash
pnpm e2e              # End-to-end tests with WebdriverIO
```

### Utilities
```bash
pnpm module-manager   # Interactive tool to enable/disable extension features
pnpm update-version <version>  # Update version across all packages
pnpm clean            # Clean all build artifacts and caches
```

## Architecture

### Monorepo Structure

**chrome-extension/** - Extension core
- `manifest.ts` - Manifest V3 configuration (permissions, content scripts, pages)
- `src/background/` - Background service worker
- `public/` - Icons and static assets

**pages/** - Extension pages (9 modules)
- `popup/` - Toolbar popup (action.default_popup)
- `options/` - Options page (options_page)
- `new-tab/` - New tab override (chrome_url_overrides.newtab)
- `side-panel/` - Side panel (side_panel.default_path, Chrome 114+)
- `content/` - Content scripts injected into web pages
- `content-ui/` - React components injected into web pages
- `content-runtime/` - Dynamically injectable scripts (from popup/background)
- `devtools/` - DevTools page (devtools_page)
- `devtools-panel/` - DevTools panel UI

**packages/** - Shared packages (12 packages)
- `shared/` - Shared types, hooks, HOCs, utilities
- `storage/` - Chrome storage API wrapper with reactive updates
- `i18n/` - Type-safe internationalization (see packages/i18n/locales/)
- `hmr/` - Custom HMR plugin for Vite
- `env/` - Environment variables (CEB_* prefix required)
- `ui/` - Shared UI components and Tailwind config merger
- `dev-utils/` - Manifest parser, logger utilities
- `vite-config/` - Shared Vite configuration
- `tsconfig/` - Shared TypeScript configuration
- `tailwindcss-config/` - Shared Tailwind configuration
- `module-manager/` - Tool to enable/disable features
- `zipper/` - Build packaging utility

### Key Patterns

**Storage Pattern** (packages/storage/)
```typescript
import { createStorage, StorageEnum } from '@extension/storage';

const myStorage = createStorage<MyType>(
  'storage-key',
  defaultValue,
  { storageEnum: StorageEnum.Local, liveUpdate: true }
);

// Usage
await myStorage.get();
await myStorage.set(newValue);
await myStorage.set(prev => ({ ...prev, updated: true }));
```

**Internationalization** (packages/i18n/)
```typescript
import { t } from '@extension/i18n';

t('keyName');                    // Simple translation
t('greeting', 'John');           // With placeholder
```
Add translations in `packages/i18n/locales/{locale}/messages.json`

**Environment Variables**
- Edit `.env` file (variables must have `CEB_` prefix)
- CLI override: `pnpm set-global-env CLI_CEB_VAR=value`
- Import constants: `import { IS_DEV } from '@extension/env'`

### Build Pipeline

1. `pnpm dev` triggers Turborepo
2. Each page package runs `vite build --mode development`
3. Output goes to `dist/` directory
4. Custom HMR plugin watches for changes and injects updates
5. Browser extension reloads via HMR WebSocket connection

### Manifest Configuration

Edit `chrome-extension/manifest.ts` to configure:
- Permissions (currently: storage, scripting, tabs, notifications, sidePanel)
- Host permissions (currently: <all_urls>)
- Content script matches and injection rules
- Extension pages registration

**Important**: The default manifest grants broad permissions. For production, restrict to only necessary permissions and hosts.

## Development Workflow

### Adding a New Page
1. Create directory in `pages/your-page/`
2. Add `package.json` with dependencies on shared packages
3. Create `src/` with React components
4. Register in `chrome-extension/manifest.ts`
5. Turborepo will automatically include it in builds

### Adding Shared Code
- **Types/utilities**: Add to `packages/shared/`
- **React components**: Add to `packages/ui/`
- **Storage implementations**: Add to `packages/storage/lib/impl/`

### Installing Dependencies
```bash
pnpm i <package> -w              # Install to root
pnpm i <package> -F <module>     # Install to specific package
```
Module name is the `name` field in package.json without `@extension/` prefix (e.g., `popup`, not `@extension/popup`)

## Troubleshooting

**HMR frozen**: Ctrl+C dev server, kill any lingering `turbo` processes, run `pnpm dev` again

**Import resolution issues in WSL**: Install "Remote - WSL" VS Code extension and connect to WSL remotely

**Chrome permissions warning**: Edit `chrome-extension/manifest.ts` to restrict `host_permissions` and `permissions` before publishing
