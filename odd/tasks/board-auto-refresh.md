# ODD Feature: Board auto-refresh

**Status:** Implemented; review and work-unit commit pending
**Branch:** `feat/board-auto-refresh`
**Scope:** GUI task synchronization only; MCP contracts stay unchanged.

## Specs

- **S1:** “La situacion es sencilla, tengo un monitor dedicado al tablero, tengo agentes que interactuan con el, pero hasta que no refresco la pagina manualmente no veo los cambios de estado, como se resuelve?”
- **S2:** “Solo lo aplicaria a la GUI, los agentes solo piden por MCP cuando necesitan.”
- **S3:** “Comienza”

## Tasks

- [x] `SYNC-01` — Map task reads, optimistic mutations, and UI consumers; confirm the Next.js 16 read mechanism. Route: exploration. Evidence: `TasksProvider` owns a stale initial snapshot; Server Action reads are the smallest Next.js 16 boundary. Commit: —
- [x] `SYNC-02` — Add RED tests for visible-tab polling, snapshot reconciliation, and pending local mutations. Route: test-first. Evidence: `npm run test:sync` fails because `lib/tasks-sync.js` is absent; contract covers replacement, deferred latest snapshot, stale threshold, and recovery. Commit: —
- [x] `SYNC-03` — Add a read-only task snapshot boundary and reconciliation helpers without changing MCP. Route: implementation. Evidence: `getTaskSnapshotAction` and pure reconciliation helper pass 6 sync tests. Commit: —
- [x] `SYNC-04` — Add visible-tab polling to `TasksProvider`, defer external snapshots during local writes, and expose passive sync state. Route: implementation. Evidence: 5-second visible-tab polling, deferred latest snapshot, silent failures, and localized stale indicator; focused tests/type-check pass. Commit: —
- [x] `SYNC-05` — Verify board, task detail, and calendar update from an external task mutation without clobbering local optimistic work. Route: verification. Evidence: Chromium against an isolated Docker image/BDD observed a direct external task mutation in board, calendar, and task-detail UI after polling; 6 pure reconciliation tests cover deferred snapshots and preservation of a local optimistic mutation without a remote snapshot. Commit: —
- [ ] `SYNC-06` — Review the diff, record evidence, and prepare a reviewable work-unit commit. Route: closure. Commit: —

## Log

- **L1:** “La situacion es sencilla, tengo un monitor dedicado al tablero, tengo agentes que interactuan con el, pero hasta que no refresco la pagina manualmente no veo los cambios de estado, como se resuelve?”
- **L2:** “Solo lo aplicaria a la GUI, los agentes solo piden por MCP cuando necesitan.”
- **L3:** “Comienza”
- **L4:** Accepted implementation direction: poll only while the GUI tab is visible; use a five-second interval, defer external snapshots while an optimistic local mutation is pending, and avoid per-refresh toasts. This keeps the MCP protocol unchanged.
- **L5:** “Perfecto, avanza con la proxima fase”
- **L6:** Accepted UX policy: polling stays silent and does not use `SyncProvider`; preserve the current indicator for GUI writes, keep the last valid board on a failed read, expose a discreet stale-data state only after three consecutive polling failures, and clear it on recovery.
- **L7:** SYNC-02 RED observed: `npm run test:sync` fails with `Cannot find module '../../lib/tasks-sync.js'` until the production helper is added.
- **L8:** SYNC-03/04 GREEN: `npm run test:sync` passes 6/6; `npm run test:i18n` passes 14/14; `npx tsc --noEmit` and `git diff --check` pass. A first implementation regression that would have replaced a successful optimistic mutation with the initial snapshot was caught and corrected before functional verification.
- **L9:** SYNC-05 smoke: a clean Docker image and seeded isolated PostgreSQL volume served the GUI on port 3002. Chromium observed external SQL mutations reflected after the polling interval in the board, calendar, and task-detail input. Test data was restored and temporary containers/volumes were removed.
