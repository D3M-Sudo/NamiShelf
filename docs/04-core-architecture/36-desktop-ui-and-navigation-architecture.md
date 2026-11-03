# NamiShelf — Desktop UI and Navigation Architecture

**Status:** Compose Multiplatform Desktop approved under ADR-014; hybrid desktop shell approved under ADR-015. Detailed responsive behavior and visual design remain open.  
**Target:** Linux desktop, keyboard and pointer input, scalable window layouts.

## 1. Product UX direction

NamiShelf uses Compose Multiplatform Desktop and a desktop-first hybrid UX inspired by Mihon, not a literal Android screen port. Linux build/runtime, memory, rendering, accessibility and package compatibility are mandatory M0 validation gates under ADR-014. The interface should support both focused reading and information-dense library management. Navigation, dialogs, menus, keyboard behavior and file workflows must follow desktop conventions.

## 2. Primary information architecture

Recommended primary destinations:

1. **Library** — all entries, categories, filters, sorting and bulk actions.
2. **Sources** — source catalog, installed/available sources and repository management.
3. **Browse** — source discovery and search.
4. **Downloads** — queue, active tasks, completed files and failures.
5. **History** — recent reading and reading activity.
6. **Tracking** — external tracking associations and status.
7. **Extensions** — installed extensions, repositories, permissions and diagnostics.
8. **Settings** — application, reader, sources, downloads, privacy, backup, sync and platform settings.

Reader is a contextual full-window/focused workspace launched from a library entry or chapter, not necessarily a permanent primary navigation destination.

Backup and Sync are cross-cutting workflows surfaced from Settings and relevant library actions, with dedicated progress/status views.

## 3. Desktop shell layout

Proposed shell:
- Collapsible persistent left sidebar with primary destinations.
- Main content region with responsive list/grid/detail layouts.
- Optional contextual detail pane on sufficiently wide windows; collapse it into a navigable detail view at narrow widths.
- Global command/search entry point.
- Status surface for downloads, sync state, extension errors and background tasks.

The hybrid shell and dedicated reader workspace are owner-approved under ADR-015. Reader entry/exit must preserve the prior navigation context; background operations continue independently. Under ADR-016, these workspaces are hosted in one top-level application window; reader mode does not spawn a separate application window.
- Standard menus, dialogs and notifications.

Avoid permanently visible dense toolbars that compete with reading content. The reader should offer a distraction-reduced mode and predictable escape/navigation behavior.

## 4. Navigation model

- Use stable typed route identifiers, not display strings or list indexes.
- Preserve navigation history for back/forward within feature workflows.
- Deep links must be validated and mapped to safe internal routes.
- Restore only non-sensitive navigation state after restart.
- Opening an external URL or file uses platform integration and explicit user action.
- Modal dialogs must not be used for workflows that need persistent progress or recovery.

## 5. Feature screen responsibilities

| Destination | Primary screens |
|---|---|
| Library | library overview, category view, entry detail, filters, bulk selection |
| Sources | source catalog, source detail, repository list |
| Browse | source selection, search results, filters, entry preview |
| Downloads | queue, task detail, completed/failed list |
| History | timeline/list, filters, entry navigation |
| Tracking | tracker status, association editor, sync action |
| Extensions | installed list, details, permissions, repository management, logs |
| Settings | grouped preferences, backup/restore, sync/devices, appearance/platform |

## 6. Reader UX architecture

The reader is a distinct interaction mode with:
- page canvas and fit/zoom controls;
- next/previous chapter navigation;
- reading direction and layout options where supported;
- page/chapter progress;
- bookmark and reader menu;
- keyboard shortcuts with discoverable help;
- optional page strip/thumbnail navigation;
- explicit loading, decode failure and offline states.

Reader settings are divided into portable defaults and device-specific rendering preferences. Reader navigation must remain usable when background sync is offline.

## 7. State ownership

- View state belongs to the screen/view-model layer and is reconstructible.
- Durable user state is written through application use cases.
- Long-running operation state is owned by its service and observed by the UI.
- Do not keep duplicate mutable copies of library/read state in screen state.
- UI must handle loading, empty, partial, stale, offline, permission-denied and error states explicitly.
- Avoid UI-triggered network calls during recomposition; actions are explicit and idempotent where possible.

## 8. Accessibility and desktop behavior

- Keyboard navigation and visible focus indicators.
- Screen-reader labels for icon-only controls.
- Adequate contrast and scalable text.
- Respect desktop font scaling and HiDPI/fractional scaling.
- Context menus and shortcuts must not be the only route to important actions.
- Avoid hover-only affordances.
- File pickers use platform portal/native integration where available.
- Window resizing and narrow widths must not hide critical actions.

## 9. Visual system

Use a shared design system for spacing, typography, semantic colors, icon sizing, elevation and component states. Avoid copying Android Material navigation patterns without adaptation. Establish light/dark theme behavior and high-contrast semantics before polishing feature screens.

## 10. Single-window lifecycle

NamiShelf has one top-level application window for release 1 (ADR-016). Feature navigation, contextual details and reader mode transition within this window. Native dialogs, menus and file pickers may use transient platform surfaces. Screen composition lifecycle must not own long-running downloads, tracking or synchronization jobs. Window size, minimum dimensions, close behavior and startup restoration remain open for M0/UX review.

## 11. Decisions to validate with prototypes

- Sidebar versus compact navigation rail behavior.
- Library density and optional detail pane.
- Global search/command palette scope.
- Reader full-window behavior and page-strip placement.
- Settings information architecture.
- Exact Compose Desktop navigation/state libraries and state-management approach (framework fixed; supporting libraries deferred).
- Minimum window size and responsive breakpoints.

These should be settled through low-cost wireframes/prototypes and owner review, not guessed as final visual decisions. The high-level layout choice itself is no longer open: sidebar collapse, conditional detail pane and dedicated reader workspace are approved.

Multi-window support is not a release-1 requirement. Detached reader windows may be reconsidered only through a future explicit architecture decision. No UI implementation or visual prototype has been created. Framework adoption is conditional on passing the Linux M0 gates; failure requires evidence-based owner review.
