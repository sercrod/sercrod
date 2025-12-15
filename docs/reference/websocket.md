# WebSocket

_What you will find:_ normative behavior for `*websocket` and `*ws-send`, including connection lifecycle, exposed state, events, and host helper APIs. No tutorials are included.

## Scope

This page defines:
- When a connection is created and closed.
- What data fields are updated on the host.
- Which events are emitted and with what timing.
- How payloads are parsed and sent.

---

## `*websocket="urlOrConfigExpr"`

- **When**
  - Evaluated on render.
  - Re-evaluated when its parameters change or the host is force-updated.
- **Action**
  - Opens a WebSocket connection.
  - If a connection is already open with different parameters, the existing one is closed and a new one is opened.
- **Value**
  - `urlOrConfigExpr` can be a string URL or a configuration object understood by the implementation. At minimum, a URL string is supported.

### Host data fields

While active, the host maintains the following reactive fields:

- `$ws_ready` - boolean, `true` when the socket is open.
- `$ws_error` - last error object or descriptive value, `null` if none.
- `$ws_last` - last received message payload. If JSON is valid, it is the parsed object, otherwise a string or binary representation as supported.
- `$ws_messages` - array of received payloads in chronological order. Implementation may cap its length.
- `$ws_closed_at` - timestamp of last close event, or `null`.
- `$ws_close_code` - numeric close code from the last close event, or `null`.
- `$ws_close_reason` - string reason from the last close event, or `""`.

These fields are updated by the directive in response to socket events and are part of the normal update cycle.

### Events

The host dispatches the following CustomEvents:

- `sercrod-ws-before-connect` - right before attempting to connect. `detail` includes `{ host, url }`.
- `sercrod-ws-open` - after the socket reaches OPEN. `detail` includes `{ host, url }`.
- `sercrod-ws-message` - on each incoming message. `detail` includes `{ host, url, raw, body }` where `body` is JSON-parsed if possible.
- `sercrod-ws-error` - on socket error. `detail` includes `{ host, url, error }`.
- `sercrod-ws-close` - on close. `detail` includes `{ host, url, code, reason, wasClean }`.

Event names and timings are stable. Payload shapes may include additional implementation-defined fields.

### Parsing of incoming messages

- If the message text parses as JSON, the directive sets `$ws_last` to the parsed object and pushes it into `$ws_messages`.
- Otherwise it uses the raw text string. Binary frames are handled according to the implementation - at minimum they are exposed as raw data objects or ArrayBuffers.

### Re-connection policy

- The directive does not imply automatic retries unless the implementation specifies otherwise.
- Programmatic reconnection can be performed through the host helper API described below.

---

## `*ws-send="payloadExpr"`

- **When**
  - Evaluated on demand - typically by an event or explicit update.
- **Action**
  - Sends the evaluated payload on the active socket associated with the host.
- **Payload rules**
  - If the value is an object or array, the implementation serializes it to JSON.
  - If the value is a string, it is sent as-is.
  - Binary payloads are supported if the value is an ArrayBuffer or a view type the implementation accepts.

If no socket is currently open, the send is a no-op or results in a warning depending on configuration.

---

## Host helper API

When `*websocket` is present, the host exposes a helper under the element for direct control:

- `el.websocket.connect(urlOrConfig?)` - opens a connection, optionally overriding parameters.
- `el.websocket.reconnect()` - closes and reopens using the last known parameters.
- `el.websocket.close(code?, reason?)` - closes the current connection.
- `el.websocket.send(value)` - sends a payload, applying the same serialization rules as `*ws-send`.
- `el.websocket.status()` - returns an object describing the current state `{ ready, url, last_url, last_error, last_close }`.
- `el.websocket.urls` - an object or record of last resolved URL values as maintained by the implementation.

Helper presence and exact shapes are stable at the names above. Additional helper fields may exist.

---

## Parameter changes and multiple hosts

- Re-evaluating `*websocket` with a new URL or configuration closes the previous connection and opens a new one, updating state fields accordingly.
- Multiple Sercrod hosts can maintain independent sockets. State is not shared across hosts.

---

## Error handling

- Errors set `$ws_error` and dispatch `sercrod-ws-error`.
- After an error or close, state fields reflect the last known values. `$ws_ready` becomes `false`.
- Sending while not ready has no effect or logs a warning.

---

## Cleanup and lifecycle

- On host disconnection, the implementation may close the socket or leave it to the browser. If closed, the standard close event is dispatched and state fields update.
- No transient keys related to WebSocket are auto-cleared in finalization, except those that are explicitly documented as transient elsewhere. WebSocket state fields persist until the next change.

---
Next page: [`world.md`](./world.md)
