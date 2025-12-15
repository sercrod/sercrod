# Warnings and Errors

_What you will find:_ normative rules for how the runtime reports warnings and errors, the events it emits, the console logging policy, and the guards that prevent runaway updates. This is a reference - no tutorials or design commentary.

## Scope

This page defines:
- Categories of failures that may occur during a render cycle or side-effect operation.
- Which CustomEvents are emitted and with what `detail` shape.
- How the per-host logging configuration affects console output.
- Deterministic timing of emission relative to the update cycle.

---

## Categories

Sercrod classifies issues into these areas for reporting:

- **http** - `*fetch`, `*post`, `*api` failures, timeouts, non-2xx responses, or response parse errors.
- **upload** - `*upload` initiation failures, transport errors, non-2xx responses, or response parse errors.
- **download** - `*download` initiation failures or transport errors.
- **websocket** - connection errors, protocol errors, unexpected closure with codes.
- **render** - failures during template rendering, directive execution, or binding application.
- **parse** - failures during template parsing, AST hook execution, or pre-hook processing.
- **other** - any unclassified issue.

Implementations may add subcodes internally but must expose the top-level `area` above when emitting `sercrod-error`.

---

## Events

### Lifecycle

#### `sercrod-updated`
- **When**: after the DOM for the cycle is committed, immediately before transient cleanup is scheduled for the next frame.
- **detail**: `{ host }`

#### `sercrod-error`
- **When**: an unrecoverable failure surfaces to the host and the operation cannot complete as intended.
- **detail**: `{ host, area, error }`  
  - `area`: one of the categories above  
  - `error`: `Error` instance or implementation-defined diagnostic object

### HTTP and API

#### `sercrod-api`
- **When**: a `*fetch`, `*post`, or `*api` operation completes successfully.
- **detail**: `{ host, url, status, body }`

- **Error path**: emits `sercrod-error` with `area: "http"` when the operation fails.

### File transfer

#### `sercrod-upload-start` / `sercrod-upload-progress` / `sercrod-uploaded`
- **When**: upload lifecycle around `*upload`.
- **Error path**: emits `sercrod-error` with `area: "upload"` when the operation fails.

#### `sercrod-download-start` / `sercrod-downloaded`
- **When**: download lifecycle around `*download`.
- **Error path**: emits `sercrod-error` with `area: "download"` when the operation fails.

### WebSocket

#### `sercrod-ws-before-connect` / `sercrod-ws-open` / `sercrod-ws-message` / `sercrod-ws-close`
- **When**: WebSocket lifecycle around `*websocket`.
- **Error path**: emits `sercrod-ws-error` and may also emit `sercrod-error` with `area: "websocket"`.

---

## Console logging policy

Each host exposes `el.error: { fatal: boolean, warn: boolean, debug: boolean }`.

- **fatal** - controls emission of serious errors to the console. Does not change event emission.
- **warn** - controls warning logs for recoverable issues (e.g., failed hooks, unsupported combinations, guard trips).
- **debug** - controls verbose logs helpful for inspection.

Notes:
- These flags influence console output only. They do not suppress CustomEvents.
- Implementations may choose additional internal verbosity levels but must honor these three booleans semantically.

---

## Guards and protections

### Re-entrancy guard
- A host never renders recursively within the same pass. Attempts to schedule nested renders are deferred to the next frame.

### Loop limit
- A single update cycle is bounded by a fixed iteration limit (100). Exceeding the limit terminates the cycle and logs a warning. The host remains operational for subsequent cycles.

### Expression exceptions
- Exceptions thrown during expression evaluation (including directive values) abort the effect of the current directive, log a warning or error according to `el.error`, and the render pass continues for other nodes.

### Hook failures
- Exceptions in AST hooks or pre-hooks result in a warning and fallback behavior:
  - AST hook failure: continue with the previous or unmodified AST for that node when possible.
  - Pre-hook failure: continue with the unmodified template input.

### Network and file failures
- Transport failures, non-2xx status (when treated as failure), and parse errors emit `sercrod-error` with the appropriate `area`.
- Where a completion event exists (e.g., `sercrod-api`/`sercrod-uploaded`/`sercrod-downloaded`), it is not emitted on failure.

### WebSocket failures
- Connection or protocol errors emit `sercrod-ws-error` and may additionally emit `sercrod-error` with `area: "websocket"`.
- After close or error, state fields reflect the last known values; `$ws_ready` becomes `false`.

---

## Data and transient stores

- Transient stores (`$upload`, `$download`, and all keys recorded via `*into`) are visible during the cycle that produced them and are cleared in the next-frame finalization.
- Implementations may expose additional diagnostic fields in host data for inspection; their presence and shape are implementation-defined and not required by this reference.

---

## Ordering guarantees

Within a successful cycle that includes side effects:

1) Side-effect directive completes and writes results to host data.  
2) Associated completion event is dispatched (`sercrod-api` / `sercrod-uploaded` / `sercrod-downloaded` / `sercrod-ws-*`).  
3) `sercrod-updated` is dispatched after the DOM commit for the cycle.  
4) On the next animation frame, transient cleanup runs and clears `$upload`, `$download`, and all `*into` keys recorded during the cycle.

On failure:
- The corresponding error event is emitted (`sercrod-error` and/or `sercrod-ws-error`), and no completion event for that operation is dispatched.

---

## Non-goals

- This page does not define how to sanitize HTML or escape attributes. See `filters.md` for the `html` and `attr` filters.
- This page does not define retry strategies for network or WebSocket operations. Implementations may provide helper APIs; policy remains implementation-defined.

---
Back to index: [`README.md`](./README.md)
