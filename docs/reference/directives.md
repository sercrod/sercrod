# Directives

_What you will find:_ a precise, alphabetical-by-category reference of every directive and shorthand Sercrod recognizes. Each entry states the value form, when it is evaluated, and what side effects it has on data, attributes, DOM, and events. No tutorials or design commentary are included.

## Conventions

- All directives have two equivalent forms: the asterisk form (`*if`) and the `n-` form (`n-if`). This page lists the asterisk form for brevity.
- Expressions are JavaScript evaluated in the current scope. In event handlers, `$event` is the native DOM event and `el` is the element that holds the handler.
- Text interpolation uses `%expr%`. Output directives control whether text or HTML is emitted.
- Transient stores: `$upload`, `$download`, and any keys named by `*into` are cleared automatically in the finalization step of the update cycle that produced them.

---

## 1) Control flow

### `*if="expr"`
- **When:** render time and on updates.
- **Effect:** renders the element and its subtree only if `expr` is truthy. Otherwise the element is removed from the rendered output.

### `*elseif="expr"`
- **When:** render time and on updates.
- **Effect:** used immediately after an `*if` or another `*elseif`. Renders the element only if all previous conditions in the same chain were falsy and `expr` is truthy.

### `*else`
- **When:** render time and on updates.
- **Effect:** final branch in an `*if` chain. Renders only if all previous `*if`/`*elseif` conditions in the same chain were falsy.

### `*switch="expr"`
- **When:** render time and on updates.
- **Effect:** activates the first matching `*case` in the same switch group, otherwise `*default` if present. Matching is strict equality (`===`) against each case value.

### `*case="valueExpr"`
- **When:** evaluated within the nearest active `*switch`.
- **Effect:** renders this element if `valueExpr === <switch-expr>`. May be combined with `.break` (see below).

### `*case.break="valueExpr"`
- **When:** same as `*case`.
- **Effect:** same as `*case`, and prevents any subsequent cases in the same switch group from rendering.

### `*break`
- **When:** inside a `*switch` group.
- **Effect:** stops further case evaluation and rendering within the same switch group.

---

## 2) Iteration

### `*each="iterableExpr as item[, index]"`  
- **When:** render time and on updates when `iterableExpr` changes.
- **Effect:** repeats the element once per entry in an array or object.  
  - For arrays: `item` is the value, `index` is the numeric index.  
  - For objects: `item` is the value, `index` is the property key.

### `*for="loopExpr"`
- **When:** render time and on updates.
- **Effect:** low-level looping. `loopExpr` must evaluate to an iterable or a structure that Sercrod can iterate. Each iteration renders one instance of the element. Use `*each` for common array/object cases.

---

## 3) Variables and scope

### `*let="assignments"`
- **Form:** one or more comma-separated assignments, e.g. `*let="x = 1, y = x + 2"`.
- **When:** evaluated before rendering the element.
- **Effect:** defines or updates variables in the current scope for this subtree. During `*let` evaluation, `$parent` is injected and points to the nearest ancestor host’s data (read-write).

### `*global="assignments"`
- **When:** before rendering the element.
- **Effect:** writes named values either to the host data if a same-named key exists, otherwise to `globalThis`. Use to expose values across hosts intentionally.

### `*literal="jsonLike"`
- **When:** before rendering the element.
- **Effect:** injects a literal object/array/value into scope without expression evaluation side effects.

### `*rem="comment"`
- **When:** parse time.
- **Effect:** documentation-only marker. Has no runtime effect.

---

## 4) Input and staging

### `*input="pathExpr"`
- **When:** render time; listens to input-like events (`input`, `change`, composition events, and click on form controls).
- **Effect:** two-way binding between a form control and `pathExpr`.  
  - Incoming values pass through `input_in` filter.  
  - Outgoing values pass through `input_out` filter.  
  - If staging is enabled (see `*stage`), writes go to the stage buffer instead of the main data.

### `*lazy`
- **When:** with `*input`.
- **Effect:** defers updates to `change` or blur-like points depending on the control type.

### `*eager`
- **When:** with `*input`.
- **Effect:** updates on `input` continuously.

### `*stage`
- **When:** before rendering the element.
- **Effect:** enables a per-host staging buffer. Subsequent `*input` writes go to the staged copy.

### `*apply`
- **When:** on evaluation.
- **Effect:** copies staged values into the main data. No effect if staging is not active.

### `*restore`
- **When:** on evaluation.
- **Effect:** discards staged changes. No effect if staging is not active.

### `*save`
- **When:** on evaluation.
- **Effect:** serializes the current data to a downloadable JSON file. Filename is implementation-defined.

### `*load`
- **When:** user selects a JSON file.
- **Effect:** reads JSON and merges values into the host data. Triggers a normal update cycle.

---

## 5) Output

### `*print="expr"`
- **When:** on render and updates.
- **Effect:** sets the element’s text content to the evaluated result of `expr` after applying text-related filters.

### `*compose="expr"`
- **When:** on render and updates.
- **Effect:** sets the element’s HTML content to the evaluated result of `expr` after applying HTML-related filters.

### `*textContent="expr"` / `*innerHTML="expr"`
- **Effect:** synonyms for `*print` and `*compose` respectively.

---

## 6) Attributes and shorthands

### `:class="expr"`  (alias: `n-class="expr"`)
- **When:** on render and updates.
- **Effect:** updates the `class` attribute. `expr` may be a string, an array of class names, or an object map `{name: boolean}`. The `attr` filter applies.

### `:style="expr"`  (alias: `n-style="expr"`)
- **When:** on render and updates.
- **Effect:** updates inline `style`. `expr` may be a CSS string or an object map `{prop: value}`. The `style` filter applies.

---

## 7) Communication (HTTP and API)

### `*fetch[="urlExpr"]`
- **When:** on render and when its parameters change or the host is force-updated.
- **Effect:** performs a GET request. If the response is JSON, stores the parsed object into the host data.  
  - Storage target: if `*into="name"` is present, result is available as `data[name]` for the current cycle and then cleared in finalization.  
  - Without `*into`, the default integration is host-defined.

### `*post="urlExpr"`
- **When:** on evaluation.
- **Effect:** performs a POST with a JSON body derived from the current data or an explicit expression. Response handling mirrors `*fetch`.

### `*api="expr"`
- **When:** on evaluation.
- **Effect:** performs an arbitrary HTTP request defined by `expr` (URL, method, headers, body). Emits completion events. Supports `*into`.

### `*into="name"`
- **When:** with `*fetch`, `*post`, `*api`, `*upload`, `*download`.
- **Effect:** designates a property in the host data that receives the operation result for the current cycle. Keys recorded via `*into` are cleared in finalization.

---

## 8) Files

### `*upload="targetExpr"`
- **When:** user selects files via the control.
- **Effect:** sends selected files using XHR with progress events.  
  - Events: `sercrod-upload-start`, `sercrod-upload-progress`, `sercrod-uploaded`.  
  - Result storage: sets `$upload` and, if `*into` is present, also writes to that key for the current cycle.

### `*download="urlOrBlobExpr"`
- **When:** on evaluation.
- **Effect:** downloads a resource or generated Blob.  
  - Events: `sercrod-download-start`, `sercrod-downloaded`.  
  - Result storage: sets `$download` and respects `*into` if present.

---

## 9) WebSocket

### `*websocket="urlExpr"`
- **When:** on render and when parameters change or the host is force-updated.
- **Effect:** opens a WebSocket connection.  
  - State keys on host data: `$ws_ready`, `$ws_error`, `$ws_last`, `$ws_messages[]`, `$ws_closed_at`, `$ws_close_code`, `$ws_close_reason`.  
  - Events: `sercrod-ws-before-connect`, `sercrod-ws-open`, `sercrod-ws-message` (payload JSON auto-detected), `sercrod-ws-error`, `sercrod-ws-close`.  
  - Host exposes methods under `el.websocket.{connect,reconnect,close,send,status,urls}`.

### `*ws-send="payloadExpr"`
- **When:** on evaluation.
- **Effect:** sends a string or JSON-serializable payload to the active socket associated with the host. If multiple sockets are managed, resolution follows the host’s current `el.websocket.status()` rules.

---

## 10) Events and handlers

### `@type="expr"`  (event handler)
- **When:** native event dispatch.
- **Effect:** evaluates `expr` in a context where `$event` is the DOM event and `el` is the current element.  
- **Modifiers:** append with dots to the event name.  
  - `.prevent` → `event.preventDefault()`  
  - `.stop` → `event.stopPropagation()`  
  - `.once` → `addEventListener(..., { once: true })`  
  - `.capture` → listener in capture phase  
  - `.passive` → passive listener  
  - `.update` → force a host update after handler execution  
  - `.noupdate` → suppress auto-update for this handler  
- **Auto-update policy:** input-like events schedule updates by default. Use `.noupdate` to opt out or `.update` to force an update on non-input events.

---

## 11) Lifecycle helpers and methods

### `*updated="handlers"`
- **When:** after initialization and after each update cycle.
- **Effect:** invokes named functions or all function properties of a named object. `handlers` can be a comma-separated list of function names or object identifiers.

### `*methods="namesOrObjects"`
- **When:** before rendering the element.
- **Effect:** injects named global functions and/or all function properties of named global objects into the expression scope of this host.

### `*log="expr"`
- **When:** on evaluation.
- **Effect:** writes the evaluated value to the console for inspection. No side effects.

### `*strict`
- **When:** before rendering the element.
- **Effect:** turns on stricter evaluation rules for this subtree as defined by the current implementation’s strict mode.

### `*prevent-default`
- **When:** on evaluation.
- **Effect:** attaches default-prevention behavior in contexts where no `@` handler is used. Prefer handler modifiers when possible.

---

## Errors and events

- Failures in network, file, or WebSocket operations emit corresponding events (`sercrod-error` and operation-specific events). Implementations may also log warnings.  
- Directives that assign to data keys update the reactive store and schedule rendering according to the host’s coalescing policy.

---
Next page: [`lifecycle.md`](./lifecycle.md)
