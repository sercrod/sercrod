# Host API

_What you will find:_ the public surface of a Sercrod host element (e.g., `<serc-rod>`): properties, methods, observed attributes, helper objects, and their behaviors. No tutorials or design commentary are included.

## Element identity

- **Custom element**: a Sercrod host is a custom element instance. The default tag is `<serc-rod>` unless auto-definition is disabled by configuration.
- **Independence**: each host maintains its own data scope, scheduler, caches, and directive state. Nested hosts are independent unless the user explicitly shares data.

## Observed attributes

The host reacts specially to changes in the following attributes:

- `data`  
  Parsed as JSON and wrapped reactively. Triggers an update pass.

- `*methods` / `n-methods`  
  Comma-separated function names and/or global object identifiers. Refreshes the set of method identifiers visible to expressions. Triggers an update pass.

- `*updated` / `n-updated`  
  Comma-separated function names and/or global object identifiers whose function properties are invoked after each update. Refreshes the callback set. Triggers an update pass.

Changes to other attributes are handled by the binding system during rendering but are not specially parsed on attribute-change alone.

## Properties

### `el.data: object`
- **Get**: returns the host’s reactive data object (Proxy-wrapped).  
- **Set**: replaces the entire data object. The value is wrapped reactively. Schedules an update.  
- **Notes**: nested objects are wrapped on demand. Mutations to reactive properties schedule updates according to the scheduler rules.

### `el.error: { fatal: boolean, warn: boolean, debug: boolean }`
- **Purpose**: controls logging and error reporting behavior for this host.  
- **Effect**: implementations consult these flags when emitting console output and handling non-fatal failures.

### `el.websocket: object | undefined`
- **Presence**: defined when `*websocket` is active on the host.  
- **Shape**:
  - `connect(urlOrConfig?)`  
  - `reconnect()`  
  - `close(code?, reason?)`  
  - `send(value)`  
  - `status()` → `{ ready, url, last_url, last_error, last_close }`  
  - `urls` → implementation-defined record of last resolved URL values  
- **See**: `websocket.md` for normative behavior.

## Methods

### `el.update(force = true): void`
- **Effect**: schedules an update pass for this host.  
- **Coalescing**: multiple calls within the same animation frame coalesce into a single render.  
- **Guards**: a re-entrancy guard prevents nested renders in one pass; an internal loop-limit prevents runaway re-renders.

## Events

A host dispatches CustomEvents for lifecycle and side-effect operations. The core set includes:

- `sercrod-updated` after each successful render cycle  
- `sercrod-api` on HTTP completion (`*fetch`, `*post`, `*api`)  
- `sercrod-upload-*` / `sercrod-downloaded` for file transfers  
- `sercrod-ws-*` for WebSocket lifecycle  
- `sercrod-error` on unrecoverable failures surfaced to the host

Full list and `detail` payloads: see `events.md`.

## Transient stores

- `$upload`, `$download` and all keys designated by `*into` are transient.  
- Visibility: they are available for the cycle that produced them.  
- Cleanup: cleared in the next-frame finalization after `sercrod-updated`.

## Scheduler behavior

- The scheduler batches updates using `requestAnimationFrame`.  
- Data writes, observed-attribute changes, certain event handlers, and results from network/file operations schedule updates.  
- Only one render runs per frame per host under normal conditions.

## World membership

- Each host belongs to exactly one world determined by its tag’s registration.  
- Registries (methods, filters, AST hooks) resolve by world, then the global layer.  
- World rules and cross-world behavior: see `world.md`.

---
Back to index: [`README.md`](./README.md)
