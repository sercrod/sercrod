# Communication

_What you will find:_ normative behavior for Sercrod’s network and file directives - `*fetch`, `*post`, `*api`, `*upload`, `*download` - plus `*into` and transient result keys. This is a reference, not a tutorial.

## Scope

This page defines:
- When an operation runs.
- What is sent, what is received, and how results are stored.
- Events emitted by each operation.
- Cleanup rules for transient values.

WebSocket is covered separately in `websocket.md`.

---

## Common concepts

### `*into="name"`
- **Purpose** - designates a property on the host data that receives this operation’s result.
- **Lifetime** - values written via `*into` are **transient**. Sercrod records the key name during the cycle and clears it in the next-frame finalization step.
- **Coexistence** - `*into` does not replace other result fields. For example, `*upload` also sets `$upload`. See each directive for details.

### Transient stores
- `$upload` and `$download` are set by file operations for the cycle that produced them. They are cleared in finalization together with any keys named by `*into`.

### Response parsing
- If the HTTP response is valid JSON, Sercrod parses it to an object. Otherwise the raw text or Blob is used according to the directive semantics.
- Parsing rules are implementation-defined, but at minimum JSON is supported.

---

## HTTP

### `*fetch="urlExpr"`
- **When** - on render and when any of its parameters change or the host is force-updated.
- **Action** - performs an HTTP GET to the evaluated URL.
- **Result storage**
  - With `*into="key"` - the parsed result is placed at `data[key]` for the current cycle.
  - Without `*into` - the default integration is implementation-defined.
- **Events**
  - `sercrod-api` - completion event with details of the response (status and body).
  - `sercrod-error` - emitted on network or parsing errors.
- **Side effects** - schedules a normal update after storing results.

### `*post="urlExpr"`
- **When** - on evaluation.
- **Action** - performs an HTTP POST to the evaluated URL.
- **Body** - implementation-defined. Typical implementations serialize the current host data or an expression-provided payload.
- **Result storage** - same as `*fetch` regarding `*into`.
- **Events**
  - `sercrod-api` on completion.
  - `sercrod-error` on failure.
- **Side effects** - schedules a normal update after storing results.

### `*api="configExpr"`
- **When** - on evaluation.
- **Action** - performs an arbitrary HTTP request. `configExpr` evaluates to a configuration object understood by the implementation (e.g., `url`, `method`, `headers`, `body`).
- **Result storage** - respects `*into` if present.
- **Events**
  - `sercrod-api` on completion.
  - `sercrod-error` on failure.
- **Side effects** - schedules a normal update after storing results.

---

## Files

### `*upload="targetExpr"`
- **When** - after the user selects files for the relevant control. The directive binds the necessary change handlers on render.
- **Action** - sends selected files using XHR with progress events. `targetExpr` typically resolves to an upload URL or a configuration structure understood by the implementation.
- **Result storage**
  - Sets `$upload` on the host data for the cycle that produced the result.
  - With `*into="key"` - also writes the result to `data[key]` for the current cycle.
- **Events**
  - `sercrod-upload-start` - before transmission begins. `detail` includes `{ host, el, files, url, with }`.
  - `sercrod-upload-progress` - progress notifications during transfer.
  - `sercrod-uploaded` - on completion. `detail` includes the parsed response body (if JSON) and status code.
  - `sercrod-error` - on failure.
- **Side effects** - schedules a normal update after storing results. `$upload` and any `*into` key are cleared in the next-frame finalization.

### `*download="urlOrBlobExpr"`
- **When** - on evaluation.
- **Action** - downloads a network resource or a generated Blob. The method (Fetch or XHR) and filename handling are implementation-defined.
- **Result storage**
  - Sets `$download` on the host data for the cycle that produced the result.
  - With `*into="key"` - also writes to `data[key]` for the current cycle.
- **Events**
  - `sercrod-download-start` - before initiating the transfer.
  - `sercrod-downloaded` - on completion.
  - `sercrod-error` - on failure.
- **Side effects** - schedules a normal update after storing results. `$download` and any `*into` key are cleared in the next-frame finalization.

---

## Errors and completion

- All operations emit a completion event (`sercrod-api`, `sercrod-uploaded`, `sercrod-downloaded`) or `sercrod-error` on failure.
- Implementations may also populate error-related fields in host data for inspection. The exact shape is implementation-defined.

---

## Interaction with the scheduler

- Operations that write to host data trigger the scheduler. Sercrod coalesces multiple triggers and performs a single render per frame.
- Finalization - which clears `$upload`, `$download`, and all keys recorded via `*into` - happens in the next frame after the update that stored them.

---
Next page: [`websocket.md`](./websocket.md)
