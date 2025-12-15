# Lifecycle

_What you will find:_ the host element’s lifecycle from initialization to finalization, including when parsing happens, when expressions are evaluated, when events fire, how updates are scheduled, and which transient values are cleared.

## Lifecycle timeline

A Sercrod host proceeds through these phases:

1. **Construction**
   - A custom element instance is created. The default tag `<serc-rod>` is auto-defined unless auto-define is disabled by configuration.
   - Internal fields are initialized, including data wrappers, template cache handles, and scheduler flags.

2. **Connection**
   - On `connectedCallback`, the host captures its initial `innerHTML` string as the template.
   - The `data` attribute is parsed if present. The host wraps the resulting object in a reactive proxy.
   - Observed attributes are read and applied:
     - `*methods` - method identifiers or objects to inject into expression scope.
     - `*updated` - callbacks to invoke after updates.
   - A first render is scheduled.

3. **First render**
   - The scheduler coalesces pending work and runs the update pass.
   - The template string is parsed into a lightweight AST. AST hooks are applied in registration order. Parsed ASTs are cached per template string.
   - The renderer walks the AST and produces DOM. During this, directives are evaluated and event handlers are bound.
   - After rendering, the host runs the finalization step and invokes `*updated` callbacks.

4. **Steady state**
   - The host reacts to data writes, attribute changes, and event handlers. Updates are batched per animation frame.
   - Network and file directives may run based on parameters and force-updates. Results can be routed into transient keys via `*into`.

5. **Disconnection**
   - When detached, built-in behavior is to keep data intact in memory and let DOM be garbage collected by the platform. Socket and XHR cleanup depend on their owning directives and the browser.

## Update triggers

An update pass is scheduled when any of the following occurs:

- A proxied data property is written.
- An observed attribute changes:
  - `data` - re-parsed and wrapped.
  - `*methods` or `*updated` - lists are refreshed.
- An event handler runs and either:
  - It is an input-like event that auto-schedules updates.
  - It uses the `.update` modifier.
  - It mutates data.
- Code calls `host.update(true)` programmatically.
- A directive with network or file behavior completes and writes back to data.

The scheduler uses `requestAnimationFrame` to coalesce multiple triggers into a single render. Re-entrancy is guarded by an internal `_updating` flag. An internal loop-limit protects against runaway re-renders in a single cycle.

## Rendering pass

A single update pass performs the following, in order:

1. **Preparation**
   - If needed, parse and cache the template’s AST.
   - Assemble the evaluation context for expressions:
     - Current reactive data object as scope.
     - Methods listed by `*methods` and any globally registered method sets.
     - For event expressions only: `el` is the handler’s element and `$event` is the native Event object.

2. **Node rendering**
   - Text interpolation replaces `%expr%` with evaluated results, subject to text-related filters.
   - Output directives apply:
     - `*print` sets text content.
     - `*compose` sets HTML.
     - `*textContent` and `*innerHTML` are synonyms for the above behaviors.
   - Attribute bindings apply, including `:class` and `:style` shorthands.
   - Input bindings apply:
     - `*input` attaches listeners and syncs values using `input_in` and `input_out`.
     - If `*stage` is active, writes go to the stage buffer instead of the main data.

3. **Side-effect directives**
   - Communication:
     - `*fetch` and `*post` perform HTTP I/O. Results are merged into data or placed under a transient `*into` key if specified.
     - `*api` performs arbitrary HTTP requests as configured. Results respect `*into`.
   - Files:
     - `*upload` uses XHR with progress events, sets `$upload`, and writes to `*into` when present.
     - `*download` saves a resource or Blob, sets `$download`, and writes to `*into` when present.
   - WebSocket:
     - `*websocket` opens a connection if not already open for the current parameters. `*ws-send` transmits payloads.

4. **Finalization**
   - The host schedules a deferred cleanup on the next frame that clears transient keys:
     - `$upload` and `$download`.
     - All property names recorded by `*into` during the cycle.
   - The host invokes `*updated` callbacks after the render is committed.

## Event handlers and modifiers

Event attributes use `@type="expr"` and support modifiers appended after the event type:

- `.prevent` - calls `event.preventDefault()`.
- `.stop` - calls `event.stopPropagation()`.
- `.once` - registers the handler with `{ once: true }`.
- `.capture` - registers the handler in capture phase.
- `.passive` - registers a passive handler.
- `.update` - forces a host update after the handler runs.
- `.noupdate` - prevents auto-update for this handler.
- Auto-update applies to input-like events by default. Other events require `.update` or manual data mutation to schedule a pass.

During handler evaluation, the following identifiers are available:
- `el` - the element that declared the handler.
- `$event` - the native DOM event object.
- The current data scope plus any injected methods.

## Staging buffer lifecycle

- `*stage` enables a per-host staging buffer. While active, `*input` writes go to the staged copy.
- `*apply` copies staged values into the main data and schedules an update.
- `*restore` discards staged changes and schedules an update.
- If staging is not active, `*apply` and `*restore` are no-ops.

## `*updated` callbacks

- The host calls functions listed by `*updated` after:
  - The first render.
  - Every successful update pass.
- `*updated` accepts a comma-separated list of global function names or object identifiers whose function properties are invoked. Invocation order follows the declaration order.

## Transient keys and cleanup

Transient stores are visible for the cycle that produced them and are cleared on the next frame:

- `$upload` and `$download` are set by file operations.
- Any key designated by `*into` is recorded and cleared.
- Clearing happens after `*updated` has been invoked, ensuring observers can inspect the results before cleanup.

## Nested hosts

- A host can be nested inside another. Each host maintains its own data scope, lifecycle, and scheduler.
- During `*let` evaluation only, `$parent` refers to the nearest ancestor host’s data and is writable.
- Updates in a child do not automatically re-render the parent unless data flows are explicitly connected.

## Attribute observation

The host observes and reacts to changes of:
- `data` - parsed and wrapped into reactive data.
- `*methods` and `n-methods` - refreshes available method identifiers.
- `*updated` and `n-updated` - refreshes the post-update callback set.

Changes to other attributes are handled by the binding system during rendering but are not part of the observed attribute list that triggers special parsing behavior.

## Errors and protections

- The update loop has a guard to stop runaway re-renders within a single cycle.
- Network, file, and WebSocket operations emit specific events on error and may set error-related fields under the host’s data for inspection.
- Warnings may be logged for unsupported patterns or hook failures. Error logging behavior can be configured by the host’s error settings.

---
Next page: [`communication.md`](./communication.md)
