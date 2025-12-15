# Reserved and Transient

_What you will find:_ a normative list of names and fields that are reserved by the runtime and a precise definition of transient values and their lifetimes. Use this page to avoid collisions and to reason about data visibility.

## 1) Reserved directive attributes

These attribute names are reserved for Sercrod and must not be repurposed for application data or other libraries on the same element. The `n-` form is equivalent to the `*` form.

- Control flow: `*if`, `*elseif`, `*else`, `*switch`, `*case`, `*case.break`, `*break`, `*default`
- Iteration: `*each`, `*for`
- Variables and scope: `*let`, `*global`, `*literal`, `*rem`, `*strict`
- Input and staging: `*input`, `*lazy`, `*eager`, `*stage`, `*apply`, `*restore`, `*save`, `*load`
- Output: `*print`, `*compose`, `*textContent`, `*innerHTML`
- Communication: `*fetch`, `*post`, `*api`, `*into`
- Files: `*upload`, `*download`
- WebSocket: `*websocket`, `*ws-send`
- Events helper: `*prevent-default`, `*prevent`
- Lifecycle/methods: `*updated`, `*updated-propagate`, `*methods`
- Shorthands: `:class`, `:style` (aliases: `n-class`, `n-style`)
- Event handlers: `@type="expr"` syntax and its modifiers (e.g., `.once`, `.prevent`, `.update`, `.noupdate`)

## 2) Reserved identifiers in expressions

The following identifiers are reserved and injected in specific contexts. Do not define application variables with the same names in ways that would rely on shadowing.

- `el` - injected **only in event handler expressions**; the element that holds the handler.
- `$event` - injected **only in event handler expressions**; the native DOM `Event`.
- `$parent` - injected **only during `*let` evaluation**; writable reference to the nearest ancestor host’s reactive data. Not present outside `*let`.
- Method identifiers declared by `*methods` - available as top-level names in the evaluation context for this host.

Notes:
- `this` is not defined by the runtime for expressions.
- Normal JavaScript global names (e.g., `Math`, `Date`) remain available unless shadowed by scope or `*methods`.

## 3) Reserved data keys on the host

Keys starting with `$` are reserved for Sercrod’s own use. Applications must not rely on their persistence or overwrite them unless explicitly documented.

### Transient file-operation keys
- `$upload` - set by `*upload` for the cycle that produced the result. **Transient** (see §4).
- `$download` - set by `*download` for the cycle that produced the result. **Transient**.

### WebSocket state keys (persistent until updated)
- `$ws_ready` - boolean
- `$ws_error` - last error or `null`
- `$ws_last` - last received payload (object/string/binary)
- `$ws_messages` - array of received payloads (implementation may cap length)
- `$ws_closed_at` - timestamp or `null`
- `$ws_close_code` - numeric code or `null`
- `$ws_close_reason` - string reason or empty string

These WebSocket fields are **not** cleared by the transient cleanup step. They persist until the next change in the socket lifecycle.

### Implementation notes
- Future versions may introduce additional `$` keys. Treat the `$*` namespace as reserved.

## 4) Transient values and lifetime rules

Transient values are visible **only during the update cycle that produced them** and are cleared in a deferred cleanup on the **next animation frame** after that cycle.

- **Produced by**:
  - File operations: `$upload`, `$download`
  - `*into="name"` on HTTP/API/File directives: writes the result to `data[name]` **for the current cycle only**
- **Cleanup timing**:
  1) Operation completes and writes to host data.
  2) Completion event is dispatched (e.g., `sercrod-api`, `sercrod-uploaded`, `sercrod-downloaded`).
  3) `sercrod-updated` fires after the DOM commit.
  4) Next frame: runtime clears `$upload`, `$download`, and every key recorded via `*into` in that cycle.

Implications:
- Code that needs to observe a transient must do so synchronously within or immediately after the completion event or the `sercrod-updated` of the same cycle.
- Do not store business state in `*into` targets unless you copy it to non-transient keys before cleanup.

## 5) Observed attributes (special parsing)

The host performs special parsing and update scheduling when these attributes change:

- `data`
- `*methods` / `n-methods`
- `*updated` / `n-updated`

These names are reserved for observation; other attributes are handled by normal binding during rendering and are not specially parsed on attribute-change alone.

## 6) Host properties and helper objects (names to avoid shadowing)

Do not attach application properties with the following names to the host element; they are part of the public surface:

- `data` - reactive data object (getter/setter)
- `error` - `{ fatal, warn, debug }` logging configuration
- `update(force?)` - schedules an update pass
- `websocket` - helper object exposed when `*websocket` is active, with methods `connect`, `reconnect`, `close`, `send`, `status`, and field `urls`

## 7) Collision guidance

- **Data keys**: avoid `$*` for application data. Prefer plain identifiers (e.g., `result`, `files`, `status`) or a namespaced object (e.g., `app.*`).
- **Directive names**: never reuse reserved directive names for non-Sercrod purposes on the same element.
- **Shorthands**: `:class` and `:style` are consumed by Sercrod; do not depend on other processors acting on the same attributes on Sercrod-managed elements.

---
Back to index: [`README.md`](./README.md)
