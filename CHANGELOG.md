# Changelog

All notable changes to the AutomationHQ Standalone Agent.

## [7.0.0] - 2026-03-16

### CI/CD — Complete Overhaul
- Consolidated 8 per-environment workflows into a single `build-signed-release.yml` with partner matrix (AHQ + Intelion)
- Switched distribution from S3/CloudFront to GitHub Releases (per-partner release repos)
- Standardized on npm everywhere (local + CI), removed bun dependency
- Added `package-lock.json` with npm caching (`actions/setup-node cache: 'npm'`) for faster CI installs
- Pinned all dependencies to exact versions (no `^` prefixes)
- All platforms use `actions/checkout@v4` with proper Git LFS support for Windows JRE binaries
- Windows: code signing via certificate store (USB thumbdrive), no more `CSC_LINK` env var
- Mac: notarization via `notarytool` (Xcode 13+), `appBundleId` sourced from electron-builder context
- Per-platform "Verify prerequisites" step installs `gh` CLI if missing
- `fail-fast: true` + `max-parallel: 1` ensures AHQ builds first; Intelion cancelled on AHQ failure
- Removed `build.ps1` wrapper (no longer needed)

### Build System
- `build.mjs` auto-detects platform via `process.platform` (no more manual target argument)
- `shell.mjs` uses `shell: true` for cross-platform spawn compatibility (Windows `.cmd` shim resolution)
- `notarize.mjs` reads `appBundleId` from `context.packager.appInfo.id` instead of `APP_ID` env var
- `electron-builder.json` generated dynamically per partner with correct `publish` config
- Enforced npm-only via `preinstall: "npx only-allow npm"`

### Dependencies
- Downgraded ESLint 10 → 9.39.4 (eslint-plugin-react-hooks doesn't support ESLint 10)
- Updated `@typescript-eslint/*` to 8.57.0
- Removed bun engine requirement from `package.json`
- Version bumped from 6.0.1 to 7.0.0

### Auto-Updater — GitHub Releases
- Switched auto-updater from S3 store URLs to GitHub Releases with per-partner repos
- Runtime updater URL derived from JWT token (no more `VITE_AGENT_STORE_URL` env var)
- Added update progress window with React component and IPC channels
- Static `dev-app-update.json` for development mode

---

## [6.0.1] - 2026-03-05

### Build Tooling
- Template-based → JSON config generation for electron-builder
- Replace `execa` with `shellInherit` for process spawning
- Use `agentBundleId-local-agent` for artifact naming
- Remove redundant partner name from artifact filename
- Skip code signing and notarization in unsigned release workflow
- Flatten `scripts/builders/` directory structure
- Add unsigned multi-platform release workflow with tag-push trigger

### Environment Variables
- Rename `VITE_`-prefixed env vars to non-prefixed names
- Remove `apiBuilderUrl` from app config, clean `env.d.ts`
- Remove `VITE_PARTNER_NAME`/`WEBSITE`/`COMPANY` from env injection
- Remove `.env.builder`, consolidate into dotenv with `dotenv` package

### Token & Identity
- Replace email with identity in JWT history to support ORGANIZATION tokens
- Use organizations API for token verification
- Key archival by `baseUrl:identity` composite key
- Show `tokenType` in configured tooltip and about dialog
- Rich config tooltip with email, target, and expiry

### UI Polish
- Smooth progress bar, prevent modal overlap
- Extract reusable `IdentityInfo` component
- Align TokenGate preview identity layout with AboutDialog
- Add clear cache options (current session or reset everything)

---

## [6.0.0] - 2026-02-28

### Phase 8: Foundation — Tech Debt & AppConfig Split
- Remove `rejectUnauthorized: false` from axios instance
- Remove dead code and fix dev dock name
- Rename `AppConfig` to `BuildConfig`, delete partner fields
- Extract `LogSendService` from main process
- Create reusable `Toast` component with auto-dismiss
- Wire `onConfigError` listener with toast display

### Phase 9: JWT Runtime URLs & Token Storage
- Expand `AgentJwtPayload` with `urlDetails` (10 service URLs), `partnerId`, `organizationId`
- Integrate `safeStorage` encryption for JWT token persistence
- Delete `buildApiServiceUrl`, migrate all consumers to `ConfigService`
- Remove `apiBaseUrl` from build config (now runtime-only from JWT)

### Phase 10: Startup Gate & Token UX
- Add `isConfigured()` guards to operational IPC handlers (no operations without valid token)
- Create `TokenGate` component with paste/preview/apply flow
- Create `ConfigStatusPanel` with expiry countdown
- Refactor `Entry.tsx` as gate-vs-dashboard orchestrator
- Replace JWKS verification with API-based token validation
- Centralize API error logging via axios interceptor
- Class-based `ApiService` with `setBaseUrl`
- Dynamic window sizing, EIO shutdown fix
- Replace `jose` with `jwt-decode`
- Add About dialog with partner branding
- Gate-aware menu, context menu, progress messages, `ServiceStatus` enum
- Per-environment folder archival with cancel button on token replace

### Quick Tasks (v6.0 cycle)
- **Quick-1**: Fix jar archival empty folder recreation
- **Quick-2**: JWT history backend + UI (storage, IPC, toggle/select/remove)
- **Quick-3**: Starting timeout and idle-stop guard for ProcessLifecycleService
- **Quick-5**: Create `setup.json` with partner overrides (later reverted to dynamic lookup)
- **Quick-6**: Remove `.runner` folder, consolidate token storage into `electron-store`
- **Quick-7**: Unsigned multi-platform release workflow
- **Quick-8**: Rename VITE-prefixed env vars, remove apiBuilderUrl from app config
- **Quick-9**: Runtime updater URL from JWT + progress window infrastructure

---

## [1.1.0] - 2026-03-02

### Phase 4: App Shell — Tray, Window, Update, Lifecycle
- Create `TrayService` with persistent tray, dynamic state icons, and context menu
- Create `WindowService` with close-to-tray behavior and show/hide lifecycle
- Create `UpdateService` with silent background download and tray notification
- Create `AppLifecycleService` with single-instance enforcement and protocol handler
- Create centralized IPC registry with typed handler groups (3 groups, 14 invoke + 9 event channels)
- Rewrite `index.ts` as 48-line composition root with constructor injection
- Add `paths` helper and `download-orchestrator` modules
- Switch to bun, upgrade all packages

### Phase 5: UI Modernization
- Apply DaisyUI 5 + React 19 updates
- Tailwind v4 migration descoped (stayed on v3)

### Phase 6: Build Consolidation & CI/CD
- Neutralize `package.json` — create static `tailwind.config.js` and `index.html`
- Convert `electron-builder.yml` to env interpolation
- Refactor `asset.builder.mjs` to env-only injection
- Create reusable workflow `build-agent.yml` with thin callers per product/environment
- Replace 6 duplicated workflows with thin callers

### Phase 7: Test Suite
- Configure Vitest with Electron mock helpers
- Add LoggerService and ConfigService unit tests
- Add ProcessLifecycleService unit tests (16 tests)
- Add JarDownloadService and IPC registry unit tests (14 tests)
- Add JWT paste → environment switch integration tests
- Add JAR download → start → health check integration tests
- **Total: 92 tests across 8 files**

---

## [1.0.0] - 2026-02-28

### Phase 1: Foundation — Shared Infrastructure
- Create shared directory with types, IPC constants, and `AppConfig`
- Install `jose`, create JWT utility, update tsconfigs and vite config
- Convert `require()` to ESM imports, remove deprecated packages
- Replace IPC string literals with constants
- Replace `import.meta.env` with `AppConfig`
- Resolve TypeScript errors in web tsconfig context

### Phase 2: JWT Configuration System
- Extend shared contracts with JWT/config IPC channels
- Create `LoggerService` with constructor injection (file + IPC + in-memory ring buffer)
- Add JWKS verification to `jwt.ts`, install `electron-store`
- Create `ConfigService` with JWT verification and persistence
- Add JWT paste UI and enhance LogsModal
- Wire ConfigService with JWT/config IPC handlers
- Connect protocol URL handler to ConfigService for JWT processing

### Phase 2.1: Gap Closure
- Wire JWT-derived base URL into axios HTTP client
- Fix `onLogStream` listener reference mismatch in preload

### Phase 3: Process Lifecycle
- Create shared process types and extend IPC channels
- Create `JarDownloadService` with HTTP Range resume download, retry, and ZIP extract
- Create `ProcessLifecycleService` with 7-state machine, health check, and crash recovery
- Wire services into composition root, extend preload bridge
