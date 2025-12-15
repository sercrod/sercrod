# Events

_What you will find:_ a consolidated, normative list of CustomEvents that Sercrod may dispatch, grouped by area. Each entry defines when it fires and what appears in `event.detail`. This page contains no tutorials.

## Conventions

- All events bubble and are composed unless otherwise specified by the implementation.
- Timestamps and additional fields may be included by the runtime; only the listed fields are guaranteed.
- `host` in `detail` refers to the Sercrod host element (e.g., `<serc-rod>`).
- Errors may also be logged to the console according to the host’s error configuration.

---

## 1) Lifecycle

### `sercrod-updated`
- **When**: after the first render and after each successful update cycle, immediately before transient cleanup is scheduled.
- **detail**: `{ host }`

### `sercrod-error`
- **When**: unrecoverable failures in directives or subsystems that surface to the host (network, file, WebSocket, parsing).
- **detail**: `{ host, area, error }`  
  - `area`: `"http" | "upload" | "download" | "websocket" | "render" | "parse" | "other"`  
  - `error`: an `Error` or implementation-defined diagnostic object

---

## 2) HTTP and API

### `sercrod-api`
- **When**: any HTTP operation (`*fetch`, `*post`, `*api`) completes successfully.
- **detail**: `{ host, url, status, body }`  
  - `url`: resolved request URL (string)  
  - `status`: HTTP status code (number)  
  - `body`: parsed JSON object if available, otherwise a string or Blob depending on implementation

---

## 3) File transfer

### `sercrod-upload-start`
- **When**: before starting an upload initiated by `*upload`.
- **detail**: `{ host, el, files, url, with }`  
  - `el`: the control element that triggered the upload  
  - `files`: `FileList` or array of `File`  
  - `url`: resolved upload endpoint (string, if applicable)  
  - `with`: implementation-defined per-request options

### `sercrod-upload-progress`
- **When**: periodically during upload.
- **detail**: `{ host, el, loaded, total }`  
  - `loaded`: bytes sent (number)  
  - `total`: total bytes (number) or `0` if unknown

### `sercrod-uploaded`
- **When**: after upload completes (success HTTP status).
- **detail**: `{ host, el, status, body }`

### `sercrod-download-start`
- **When**: before initiating a download via `*download`.
- **detail**: `{ host, el, url, filename }`  
  - `filename`: suggested file name (string) if available

### `sercrod-downloaded`
- **When**: after download completes (success HTTP status or Blob processing).
- **detail**: `{ host, el, status, size }`  
  - `size`: downloaded size in bytes (number) if known

---

## 4) WebSocket

### `sercrod-ws-before-connect`
- **When**: right before attempting to open a WebSocket for `*websocket`.
- **detail**: `{ host, url }`

### `sercrod-ws-open`
- **When**: after the socket transitions to OPEN.
- **detail**: `{ host, url }`

### `sercrod-ws-message`
- **When**: on each incoming message frame.
- **detail**: `{ host, url, raw, body }`  
  - `raw`: the raw message text or binary payload  
  - `body`: JSON-parsed object if `raw` is valid JSON, otherwise `undefined`

### `sercrod-ws-error`
- **When**: on WebSocket error.
- **detail**: `{ host, url, error }`

### `sercrod-ws-close`
- **When**: when the socket closes.
- **detail**: `{ host, url, code, reason, wasClean }`

---

## 5) Diagnostics and utilities

### `sercrod-log`
- **When**: when `*log="expr"` is evaluated (optional, implementation-defined).
- **detail**: `{ host, value }`

---

## Notes on ordering and cleanup

- `sercrod-updated` is dispatched after the DOM for the cycle is committed, and before transient cleanup runs on the next frame.
- Transient keys (`$upload`, `$download`, and all `*into` names recorded during the cycle) are cleared during the next-frame finalization and are available to event handlers observing the immediate `sercrod-*-completion` events.

---
Back to index: [`README.md`](./README.md)
