# Changelog

All notable changes to the AutomationHQ Standalone Agent.

## [7.0.1] - 2026-03-28

### Features
- Appium mobile automation support with full frontend integration
  - AppiumService with state machine (Idle/Starting/Running/Stopping/Error), health polling, and 60s startup timeout
  - Server start/stop/health check with configurable port (1024-65535)
  - Environment diagnostics via `appium driver doctor` (replaces deprecated `@appium/doctor`)
  - Connected device listing with model names (Android via ADB, iOS via xcrun on macOS)
  - Clickable device rows with detail modal showing Name, ID/UDID, Status, Platform with copy buttons
  - Installed driver listing with version info
  - Node.js prerequisite detection and status display
  - Appium bundled at build time via `extraResources` (no user `npm install` required)
  - Build scripts (`build.mjs`, `dev.mjs`) auto-install appium + drivers into `extra-resources/appium/`
  - Mobile toggle in Services header — enables/disables Appium, stops server on disable
- Multi-service startup UI replacing single radial progress
  - Side-by-side service cards with SVG radial progress rings, percentage display, and status labels
  - Per-service Start/Stop/Restart controls (hover-revealed) for both JAR and Appium
  - Settings icon to open Appium Setup panel
  - Health indicator dots on running services
- Stop Service IPC (`config:stopService`) for graceful JAR shutdown from UI
- Shared `shellExec` utility with exit code tracking and optional timeout (extracted from ProcessLifecycleService)
- Shared SVG icon components (`Icons.tsx`) — CheckIcon, CloseIcon, PlayIcon, StopIcon, RestartIcon, etc.
- Edit menu always visible (fixes Cmd+C/Ctrl+C not working)
- Right-click context menu on selected text (not just editable fields)
- Text selection enabled globally (`user-select: text`)
- Add Azure fallback for updates and Intelion Azure sync workflow
- Add release overwrite approval via GitHub issue
- Save and restore window size when opening/closing logs or about views (new `getWindowSize` IPC channel)
- Add per-partner default JWT tokens via workflow dispatch to pre-populate token history on first launch
- Split CI build workflow into separate AHQ and Intelion files with concurrency groups, workspace cleanup, and job timeouts
- Remove unnecessary Apple secrets from Windows/Linux CI builds

### Security
- Validate Appium server address against allowlist (127.0.0.1, localhost, 0.0.0.0, ::1)
- Validate basePath against safe regex pattern
- Runtime platform validation in `runDoctor` (prevents shell injection via IPC)
- Strict PID regex in port kill (digit-only extraction)
- Port validation on health check (1024-65535 range guard)

### Bug Fixes
- Fix doctor showing "OK" when command not found (now checks exit code + empty output)
- Fix Start-after-Stop progress: fake progress smoothly fills to ~90%, completes to 100% only when service reports running
- Fix Stop button showing "Starting" state (added Stopping/Stopped states to ServiceStatus enum)
- Handle `idle` and `stopping` process states in renderer (were unhandled, caused UI hang)
- Persist Appium server state across panel close/reopen (fetches current state on mount)
- Fix badge vertical misalignment in Appium Server control card
- Fix Appium card showing "Running" when server not started (now tracks actual server state)
- Fix duplicate percentage display in service cards (removed from status label, kept in radial ring)
- Remove all inline SVGs — consolidated into shared `Icons.tsx` (MinusIcon, SettingsIcon, CopyIcon, AlertTriangleIcon added)
- Fix macOS JRE path: remove `Home/` prefix (Azul JRE 17 tar.gz extracts flat, unlike old JRE 21 bundle)
- Fix `APPIUM_HOME` not set: drivers were installed to `~/.appium/` instead of bundled directory
- Fix xcuitest doctor timeout (10s → 30s, xcuitest checks take ~10s)
- Set `JAVA_HOME` to bundled JRE for appium commands (fixes doctor `JAVA_HOME is NOT set` warning)
- Pass `APPIUM_HOME` + `JAVA_HOME` env to all appium shell commands and server spawn
- Fix JRE download in CI: use GitHub API with token instead of `gh` CLI (not available on self-hosted runners)
- Fix CI downloading all 5 JREs instead of only the platform-relevant ones (e.g. macOS now downloads only mac_x64 + mac_arm64)
- Use hex colors for modal background to avoid oklch compatibility issue
- Use correct DaisyUI v5 CSS variable names for modal styles
- Fetch version before showing "Up to Date" dialog
- Force native light theme on title bar and window chrome for unified design across all OS modes
- Replace drop-shadow with subtle backdrop for dark mode logo visibility
- Restart process after failed token replacement or history select
- Add app name and version to native dialogs for branding
- Align identity field labels with values properly
- Await `invalidCache` calls so Reset Everything completes properly
- Prevent header divider line from breaking on menu hover
- Scale main container to fill window when resized larger
- Retain user-expanded window size when navigating between pages
- Hide Edit menu on execution screen, show only on token paste
- Show About dialog when on Replace API Token page
- Remove unnecessary horizontal scrollbar
- Make error message fully visible when replacing API token
- Disable resize on token textarea to prevent UI misalignment
- Improve log send success message with clear confirmation
- Set `package.json` description per partner during build

### Build System
- JRE 17 downloaded from GitHub Release at dev/build time instead of stored in git (saves ~738MB from repo)
  - `npm run jre:upload` — downloads JRE 17.0.18+8 from Azul CDN, uploads as zip assets to `jre-17` release tag
  - Dev downloads only current platform JRE, CI downloads all platforms
  - Supports 5 platforms: mac_x64, mac_arm64, win_x64, linux_x64, linux_arm64
- Reorganized `extra-resources/` directory structure: `extra-resources/jre/` and `extra-resources/appium/`
- Moved setup scripts to `scripts/setup/` subfolder (appium-setup, jre-setup, asset-setup)
- macOS now resolves correct JRE for Apple Silicon (`mac_arm64`) vs Intel (`mac_x64`)

### Refactor
- Extract `shellExec` into shared `src/main/utils/shell-exec.ts` (DRY: was duplicated in ProcessLifecycleService and AppiumService)
- Extract duplicate SVG icons into shared `Icons.tsx` component
- Consolidate `mobileEnabled`/`showAppium` state into `isMobileEnabled`/`isAppiumPanelOpen`
- Remove deprecated `@appium/doctor` dependency, use built-in `appium driver doctor`
- Self-explanatory naming across all new components (`ServiceCardStatus`, `jarProgressPercent`, `handleStartServer`, etc.)
- Extract `APP_NAME` constant to DRY up env var access

---

## [7.0.0] - 2026-03-16

### CI/CD — Complete Overhaul
- Consolidated 8 per-environment workflows into a single `build-signed-release.yml` with partner matrix (AHQ + Intelion)
- Switched distribution from S3/CloudFront to GitHub Releases (per-partner release repos)
- Standardized on npm everywhere (local + CI), removed bun dependency
- Added `package-lock.json` with npm caching for faster CI installs
- Pinned all dependencies to exact versions (no `^` prefixes)
- All platforms use `actions/checkout@v4` with proper Git LFS support
- Windows: code signing via certificate store (USB thumbdrive), no more `CSC_LINK` env var
- Mac: notarization via `notarytool` (Xcode 13+), `appBundleId` sourced from electron-builder context
- electron-builder publishes directly to GitHub Releases (removed manual upload steps)
- Pre-check job blocks builds if release exists unless "overwrite" is enabled
- Post-publish job syncs CHANGELOG and release notes to partner repos

### Build System
- `build.mjs` auto-detects CI via `process.env.CI` and passes `--publish always`
- `shell.mjs` uses `shell: true` only on Windows (avoids Node deprecation warning)
- `notarize.mjs` reads `appBundleId` from `context.packager.appInfo.id` instead of `APP_ID` env var
- `electron-builder.json` generated dynamically per partner with correct `publish` config
- Enforced npm-only via `preinstall: "npx only-allow npm"`
- JRE permissions auto-fixed before build (Azul Zulu ships read-only .jsa files)

### JRE & Platform Support
- Upgraded all bundled JREs to Azul Zulu 21.0.10 (was OpenLogic JDK 17 on mac, Temurin 21 on win)
- macOS: x64 build with `jre/mac_x64` (works on Intel natively, Apple Silicon via Rosetta 2)
- Windows: x64 + arm64 JREs (`jre/win_x64`, `jre/win_arm64`), dropped 32-bit support (Electron dropped 32-bit Windows)
- Linux: x64 + arm64 JREs (`jre/linux_x64`, `jre/linux_arm64`)
- Runtime JRE selection based on `process.arch` on Windows and Linux
- Added `latest-mac.yml` + `.zip` target for macOS auto-updates (pkg alone doesn't generate update metadata)
- Upload blockmap files for delta updates on all platforms

### Dependencies
- Upgraded Electron 40 → 41, vitest 4.0 → 4.1, commitlint 20.4 → 20.5
- Downgraded ESLint 10 → 9.39.4 (eslint-plugin-react-hooks doesn't support ESLint 10)
- Updated `@typescript-eslint/*` to 8.57.0
- Removed bun engine requirement from `package.json`

### Auto-Updater — GitHub Releases
- Switched auto-updater from S3 store URLs to GitHub Releases with per-partner repos
- Runtime updater URL derived from JWT token (no more `VITE_AGENT_STORE_URL` env var)
- Added update progress window with React component and IPC channels
- Static `dev-app-update.json` for development mode

---

## [6.1.0] - 2026-03-09

### Build Tooling
- Remove redundant partner name from artifact filename
- Drop `CSC_LINK` env var from CI (certificate store only)

---

## [6.0.1] - 2026-03-08

### Build Tooling
- Template-based → JSON config generation for electron-builder
- Replace `execa` with `shellInherit` for process spawning
- Use `agentBundleId-local-agent` for artifact naming
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

## [6.0.0] - 2026-03-03

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
- Fix jar archival empty folder recreation
- JWT history backend + UI (storage, IPC, toggle/select/remove)
- Starting timeout and idle-stop guard for ProcessLifecycleService
- Remove `.runner` folder, consolidate token storage into `electron-store`
- Unsigned multi-platform release workflow
- Rename VITE-prefixed env vars, remove apiBuilderUrl from app config
- Runtime updater URL from JWT + progress window infrastructure

---

## [5.4.8] - 2026-02-19

_Complete architectural rewrite (Phases 1–7). Internal milestones v1.0 and v1.1._

### Phase 1: Foundation — Shared Infrastructure
- Create shared directory with types, IPC constants, and `AppConfig`
- Install `jose`, create JWT utility, update tsconfigs and vite config
- Convert `require()` to ESM imports, remove deprecated packages
- Replace IPC string literals with constants
- Replace `import.meta.env` with `AppConfig`

### Phase 2: JWT Configuration System
- Create `LoggerService` with constructor injection (file + IPC + in-memory ring buffer)
- Add JWKS verification to `jwt.ts`, install `electron-store`
- Create `ConfigService` with JWT verification and persistence
- Add JWT paste UI and enhance LogsModal
- Wire ConfigService with JWT/config IPC handlers
- Connect protocol URL handler to ConfigService for JWT processing
- Wire JWT-derived base URL into axios HTTP client

### Phase 3: Process Lifecycle
- Create `JarDownloadService` with HTTP Range resume download, retry, and ZIP extract
- Create `ProcessLifecycleService` with 7-state machine, health check, and crash recovery
- Wire services into composition root, extend preload bridge

### Phase 4: App Shell — Tray, Window, Update, Lifecycle
- Create `TrayService` with persistent tray, dynamic state icons, and context menu
- Create `WindowService` with close-to-tray behavior and show/hide lifecycle
- Create `UpdateService` with silent background download and tray notification
- Create `AppLifecycleService` with single-instance enforcement and protocol handler
- Create centralized IPC registry with typed handler groups (3 groups, 14 invoke + 9 event channels)
- Rewrite `index.ts` as 48-line composition root with constructor injection
- Add `paths` helper and `download-orchestrator` modules

### Phase 5: UI Modernization
- Apply DaisyUI 5 + React 19 updates
- Tailwind v4 migration descoped (stayed on v3)

### Phase 6: Build Consolidation & CI/CD
- Neutralize `package.json` — create static `tailwind.config.js` and `index.html`
- Convert `electron-builder.yml` to env interpolation
- Refactor `asset.builder.mjs` to env-only injection
- Create reusable workflow `build-agent.yml` with thin callers per product/environment

### Phase 7: Test Suite
- Configure Vitest with Electron mock helpers
- Add LoggerService, ConfigService, ProcessLifecycleService unit tests
- Add JarDownloadService and IPC registry unit tests
- Add JWT paste → environment switch integration tests
- Add JAR download → start → health check integration tests
- **Total: 92 tests across 8 files**

---

## [5.4.7] - 2026-01-30

- UI changes (AHQ_D-1684)

---

## [5.4.6] - 2026-01-30

- UI changes and build fixes

---

## [5.4.5] - 2026-01-30

- CI/CD: Linux agent for UTAP, ECR image verification GHA
- Partner Intelion prod agent GHA updates

---

## [5.4.4] - 2026-01-28

- CI/CD: Push selenium image GHA
- Partner Intelion build updates

---

## [5.4.3] - 2026-01-27

- UI changes (AHQ_D-1887)

---

## [5.4.1] - 2026-01-02

- UI changes (AHQ_D-966)
- Partner test GHA build update
- Cleanup and exit when catch

---

## [5.4.0] - 2025-12-30

- Switched from bun to npm
- Partner Intelion path updates
- Devtools documentation update

---

## [5.3.6] - 2025-12-11

- Protocol name change (AHQ_D-1778)
- Partner Intelion Linux build agent GHA

---

## [5.3.5] - 2025-12-06

### Downloads & Process
- Resumable download of JAR file with graceful handling
- EBUSY removal fix (AHQ_D-1712)
- Version scanning from ZIP

### UI
- Agent download progress display
- Restart button
- Logs success/error display, JAR failure showing
- Current agent name in title and tooltip

### Build
- macOS PKG format builder
- Schemas fix for protocols
- Package upgrades

---

## [5.3.2] - 2025-11-21

- Separate protocols for different builds
- Disable auto-update of agent itself

---

## [5.3.1] - 2025-11-21

### Features
- Re-download stop (AHQ_D-1562)
- Update available failure screen update (AHQ_D-995)
- Logs modal stream + download log file
- Chrome driver from resourcesPath

### Build & CI
- Unified build process across all environments
- Certificate signing via `CSC_IDENTITY_SHA1`
- Trivy security scan + Gitleaks scan
- Linux agent builds for all environments

### Fixes
- Electron logs directory fix
- Zip downloader multiple-times fix
- Modal CSS fix (dialog to div migration)

---

## [5.2.6] - 2025-10-18

- Self-certification rejection
- Partner store URL from env
- Check for Updates text change
- Added environments configuration
- CI/CD: UTAP/Intelion GHA updates

---

## [5.2.5] - 2025-10-05

- One-click installer
- Check for Updates feature
- Then-chain to await/async refactor
- Rehash script for Windows
- Partner store URL from env

---

## [5.2.4] - 2025-09-27

- JAR download failure management (AHQ_D-1164)
- License updates (AHQ + Intelion)
- Unpackaged folder location change to userData
- Auto update for agent
- Provider URL fixes

---

## [5.2.3] - 2025-09-27

- Reconnecting ping in catch
- UTAP build actions

---

## [5.2.2] - 2025-09-24

### Features
- Auto-update feature (electron-updater manual implementation)
- Create window only when agent boots ready (AHQ_D-816)
- Hide YouTube intro (AHQ_D-1140)
- Clear cache button text change (AHQ_D-798)

### Build & CI
- UTAP Mac agent GHA
- Updated npm/node to specific versions
- Using `execa` for process spawning
- Updated engine versions

### Fixes
- Various bug fixes (fed-878, fed-859)

---

## [5.2.0] - 2025-05-17

- Version bump only (no additional changes from 5.1.1)

---

## [5.1.1] - 2025-05-17

- Electron logger wrapped as log
- DigiCert signing integration
- Dynamic URL for build and project usages (AHQ_D-398)
- Intelion signed agent push
- Axios upgrade to latest
- Various GHA build action updates

---

## [5.1.0] - 2024-11-02

### Features
- System tray implementation (nativeImage, title, exit code)
- JWT token handling via URL handler (launch storage, params)
- Local agent script builder process
- Window positioning: primary display right-bottom corner

### Build
- Latest assets builder
- Platform-specific icon handling
- dotenv with CLI for notarize CJS
- Height/width by percentage
- No suffixes for Windows compatibility

---

## [5.0.1] - 2024-07-20

- API test 3 endpoints
- Linux build: tar.gz only
- Notarize CJS fix

---

## [5.0.0] - 2024-07-14

- Updated with API 2.0 changes
- Added bun, fixed notarization
- Fix: busy jars await before delete
- Using `rmSync` for forceful deletion

---

## [4.0.1] - 2024-02-09

- Config clear feature
- Git LFS for large binary files (JRE)
- Added JRE for Linux and Mac
- Removed appImage, added deb package format
- Build resource updates
- JRE update
- Env mode refactoring (cross-env)
- Fix: operational flickering, download percent display

---

## [4.0.0] - 2023-06-22

_Complete rewrite from Angular to React + electron-vite._

- Migration to React + Electron with electron-vite
- Electron logger, extra resourcePath
- Kill port generic function (platform-wise)
- Reset log after sending
- New icon set, electron-builder YAML updates
- Notarization via `@electron/notarize`
- Moved from pnpm to npm
- Java spawning error handling improvements

---

## [3.1.0] - 2023-04-02

- Removed Selenium process (direct JAR execution)
- Added exit button

---

## [3.0.0] - 2022-12-29

- Separated starting service progress circle (AHQDEV-1293)
- Service status checking every 1-5 minutes (AHQDEV-1251)
- UI changes for local agent layout
- Maximum window disable
- Prerequisite fail/restart behind the scene
- Fix update button behavior

---

## [2.0.0] - 2022-10-18

- Protocol URL handler (`ahq-agent://`)
- Custom deep link integration
- JRE bundling
- Agent update mechanism
- Code signing (Windows + macOS notarization)
- Progress bar UI, version display
- Try-catch error handling improvements
- Various UI redesigns (download, restart pages)

---

## [1.0.3] - 2022-04-03

_Initial release — Angular-based Electron app._

- Initial Angular + Electron application
- Multi-platform installer setup (Windows, macOS, Linux)
- Test-bot tab feature, start/stop functionality
- Dev/prod mode URL switching
