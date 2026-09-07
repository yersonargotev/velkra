# Local-first data architecture for Velkra v1

Research date: 2026-09-07

## Decision

Use a **TanStack Start PWA as the application shell, PowerSync's JavaScript Web SDK as the on-device SQLite and synchronization layer, and PostgreSQL as the authoritative cloud database**. Keep Drizzle, but use it through PowerSync's official client driver in the browser and through a PostgreSQL driver in the server application. Start with managed PowerSync rather than self-hosting it.

This is a decision about the v1 architecture, not authorization to implement it without a vertical proof first. The first implementation ticket should prove, on the actual Android devices, that an installed build can create a Sale while fully offline, survive an app/browser restart, reconnect, preserve concurrent Sales from two devices, and surface a Stock discrepancy.

Do **not** use Turso Cloud embedded replicas for this PWA, and do not assume that choosing SQLite or Drizzle supplies synchronization. Reconsider Turso Sync only if Velkra moves to a native Node/Electron runtime or its TypeScript sync SDK adds a supported browser persistence/runtime path.

## Why this fits Velkra

PowerSync's Web SDK makes reads and writes against local SQLite whether the device is online or offline. It records local writes in an upload queue and uploads them through an application-defined backend connector after connectivity returns ([JavaScript Web SDK](https://docs.powersync.com/client-sdks/reference/javascript-web), [app backend setup](https://docs.powersync.com/configuration/app-backend/setup)). This directly satisfies the non-negotiable requirement that a Seller can continue operating during connectivity failures.

The local database uses IndexedDB by default and PowerSync documents browser-compatible OPFS alternatives, including an OPFS implementation intended for Safari/iOS multi-tab use. The SDK runs database work in workers and supports React hooks and TanStack integrations ([JavaScript Web SDK](https://docs.powersync.com/client-sdks/reference/javascript-web)). This is a better-supported browser path than treating a server-side SQLite file as though a PWA could open it.

PowerSync publishes an official `@powersync/drizzle-driver` for JavaScript Web and React Native. A Drizzle schema can be converted into the client schema, subject to PowerSync's SQLite type limitations ([PowerSync Drizzle integration](https://docs.powersync.com/client-sdks/orms/js/drizzle)). Thus Drizzle remains useful, but it is an access/schema tool rather than the owner of replication.

PostgreSQL is the server source of truth. TanStack Start can host the narrow authenticated endpoints that accept queued mutations and enforce domain rules; its server routes are HTTP endpoints intended for cases called from outside the Start application ([TanStack Start server routes](https://tanstack.com/start/latest/docs/framework/react/guide/server-routes)). The synchronization concern remains behind an adapter so the UI and domain commands do not depend directly on PowerSync APIs.

## Required write model

Velkra must not rely on generic last-write-wins for business events. PowerSync intentionally leaves conflict policy to the backend; its default/simple behavior is per-field last-write-wins, and its upload operations may be delivered more than once, so backend processing must be idempotent ([PowerSync conflict handling](https://docs.powersync.com/handling-writes/handling-update-conflicts)).

Use these rules:

- Give every operation a client-generated UUID and retain its device ID, actor ID, device timestamp, server receipt timestamp, and sync status.
- Model Sales, Sale Lines, Installments, Purchases, Cash Movements, Stock movements, cancellations, and corrections as append-only business events after confirmation. Never synchronize a mutable aggregate total as the sole record.
- Make backend command application idempotent by operation UUID. A retry must return the already-applied result rather than create a duplicate Sale or payment.
- Derive Stock and balances from movements or from rebuildable projections. If two offline devices sell the last unit, accept both Sales, retain both Stock movements, project negative available Stock, and create an explicit reconciliation item for the Administrator.
- Restrict last-write-wins to low-risk descriptive fields such as a note. For state transitions and financial/Stock changes, validate the expected prior state or use append-only commands and record a rejected/needs-review result instead of silently overwriting data.
- Expose `local`, `uploading`, `synced`, and `needs attention` states in the UI. “Saved locally” must never be presented as “backed up in the cloud.”

## Recovery and operational constraints

A PWA has two independent offline requirements: the application shell must load without a network, and business data must persist locally. TanStack Start supplies the React/full-stack application shape, but the build still needs a service worker and precache strategy; database sync alone does not cache the application executable.

Browser storage is not the backup. Request persistent storage where supported, keep unsynced-operation visibility, warn before logout/device reset when uploads remain, and restore a device from PostgreSQL through a full resync. Test storage eviction, interrupted uploads, duplicate delivery, schema upgrades, clock skew, token expiry while offline, and a device that reconnects after a long interval.

PowerSync's managed service adds an external operational dependency and requires a backend mutation API. Its documented architecture deliberately routes queued writes through that backend so authorization, validation, and conflict behavior remain under application control ([app backend setup](https://docs.powersync.com/configuration/app-backend/setup)). That extra component is justified here because writing and maintaining a correct browser replication protocol would be a larger and riskier system.

## Options evaluated

### Proposed SQLite + Turso + Drizzle

The current proposal is not one coherent browser architecture:

- Turso's legacy embedded replicas normally send writes to the remote primary and only work where a filesystem is available; they are not the right basis for an offline-writing PWA ([Turso embedded replicas](https://docs.turso.tech/features/embedded-replicas/introduction)).
- Modern Turso Sync does support true local writes with explicit `push()`/`pull()` and documents “last push wins” conflicts ([Turso Sync usage](https://docs.turso.tech/sync/usage)). However, Turso's TypeScript reference currently lists `@tursodatabase/sync` as native Node.js, with no ORM support, while its browser-capable/serverless package is remote-only; `@libsql/client/web` cannot open local file URLs ([Turso TypeScript reference](https://docs.turso.tech/sdk/ts/reference)). That breaks the intended installed-PWA runtime.
- “Last push wins” is too coarse for Stock and money unless Velkra builds a separate append-only command and reconciliation protocol. At that point Turso is only storage transport, while much of the hard sync work remains application-owned.

Turso remains credible for a future native application or server-side database, but not as the selected v1 browser synchronization layer.

### RxDB

RxDB is a maintained TypeScript local-first document database with IndexedDB/OPFS storage, checkpoints, replication against arbitrary backends, and custom conflict handlers ([RxDB replication protocol](https://rxdb.info/replication.html)). It can satisfy the runtime requirements, but Velkra would have to implement and operate both sides of the replication protocol and map a relational sales/Stock model into document schemas. It is the fallback if PowerSync cannot pass the device proof or its service terms are unacceptable, not the first choice.

### ElectricSQL

The earlier bidirectional SQLite/Postgres system is explicitly documented as legacy while the current system is being developed separately ([legacy ElectricSQL architecture](https://legacy.electric-sql.com/docs/reference/architecture)). A legacy bidirectional client is not an appropriate foundation for a new v1, and the current product would require additional write-path design. Do not select it without a fresh evaluation of the maintained current write story.

### Hand-built IndexedDB/outbox

IndexedDB plus an application outbox could minimize vendors for two devices, but it makes Velkra responsible for checkpoints, resumable delivery, deduplication, pull ordering, multi-tab coordination, migrations, and conflict reporting. Those are exactly the failure-prone capabilities the product requires. It is simpler in dependency count, not in total system complexity.

## Boundaries left for later decisions

This decision does not select a PostgreSQL host, authentication provider, PWA service-worker package, receipt peripheral strategy, monitoring plan, backup retention, or PowerSync pricing tier. Those choices should be made after the offline vertical proof and device questionnaire.

It also does not settle each domain conflict. The product specification must enumerate command-specific outcomes, especially for concurrent Stock, Layaway cancellation, Installments, price authorization, Daily Close, and Payables.

## Acceptance evidence for the vertical proof

Record reproducible evidence for all of the following before expanding the build:

1. First install and authenticated bootstrap while online.
2. App relaunch in airplane mode with product search and local data intact.
3. A complete Sale and Cash Movement created offline and visible after process restart.
4. Two devices independently selling the same last unit offline; both Sales remain after synchronization and an Administrator sees the discrepancy.
5. Network loss during upload and repeated upload do not duplicate any operation.
6. Logout, token expiry, and re-authentication do not discard unsynced operations.
7. A clean device can rebuild its state from the cloud after local storage is removed.
