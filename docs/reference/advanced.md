# Advanced

_What you will find:_ normative details for hooks, filters, method injection, evaluation, logging, configuration switches, and internal caches. This page specifies observable behavior and extension points without tutorials.

## AST subsystem

### Parse and cache
- The host captures its initial `innerHTML` as the template string.
- The template is parsed into a lightweight AST used by the renderer.
- Parsed ASTs are cached by template string. Reusing the same template string reuses the cached AST.

### Hook execution
- After parse and before rendering, Sercrod runs registered AST hooks in registration order.
- A hook receives the AST and may modify nodes or metadata. The hook must return either the same AST node or a replacement acceptable to the renderer.
- Hook failures do not crash the host by default - the implementation logs a warning and continues with the previous AST state when possible.

### Registration
- Implementations expose an API such as `Sercrod.register_ast_hook(fn)` that registers a function for the current world.
- Implementations may also read pre-registered hooks from an ambient array (for example `window.__Sercrod_ast_hooks`) during initialization.
- World scoping applies - hooks are registered to the current world unless an explicit global registration API is used.

### Caching and invalidation
- AST caches are world-scoped. Invalidations in one world do not affect caches in another.
- A host invalidates its cached AST when its template string changes.

## Filters

### Built-ins
Implementations ship a default set of filters and apply them at fixed points:
- `text` - used for text interpolation when converting values to strings.
- `html` - used for HTML output when composing markup.
- `placeholder` - used for placeholder text expansion.
- `attr` - used for attribute values.
- `style` - used for `style` values from strings or object maps.
- `input_in` - converts incoming control values to data.
- `input_out` - converts data values to control values.
- `url` - normalization for URL-like values where applicable.

### Registration and precedence
- Filters can be overridden per world. Lookup order is: world registry, then global layer, then implementation defaults.
- Replacing a filter changes behavior only for the world in which it was registered unless an explicit global registration is used.

### Invocation points
- Text interpolation `%expr%` uses `text` and `placeholder`.
- `*print` uses `text`. `*compose` uses `html`.
- Attribute bindings use `attr`. `:style` uses `style`.
- `*input` round-trips through `input_in` and `input_out`.

## Method injection

### Purpose
- Methods make named functions available to expressions without storing functions on the data object.

### Declaration forms
- `*methods="fnA, fnB"` exposes global functions with those identifiers.
- `*methods="SomeObject"` exposes all function properties of `window.SomeObject`.
- The `n-` prefixed form is equivalent.

### Lookup order
- Expression resolution searches:
  1) names listed by the host’s `*methods`
  2) any globally registered method sets for the current world
  3) ambient JavaScript scope as usual

### Shadowing
- Host-declared method names shadow global registrations with the same name inside that host’s expressions.

## Expression evaluation

### Scope
- Expressions are evaluated against the host’s reactive data object. The scope is the primary namespace.
- During event handler evaluation, `el` is the current element and `$event` is the native Event.
- During `*let` evaluation, `$parent` points to the nearest ancestor host’s data and is writable for that evaluation.

### Semantics
- Expressions are JavaScript snippets and may read or write scope properties.
- Writes to reactive properties schedule updates according to the scheduler rules.
- Throwing an exception logs a warning or error and may abort the current directive’s effect. The rest of the render pass proceeds.

## Logging and errors

### Error events
- Network and file operations emit completion events and `sercrod-error` on failure.
- WebSocket operations emit `sercrod-ws-*` events for open, message, error, and close.
- Implementations may attach additional details to event `detail` objects.

### Console logging
- Implementations expose a per-host error configuration object such as `{ fatal, warn, debug }`.
- Warnings are emitted for situations like excessive update loops, failed hooks, or unsupported directive combinations.
- `*log="expr"` evaluates and logs the result without side effects on data or DOM.

## Scheduler and guards

- The scheduler batches updates with `requestAnimationFrame`.
- Re-entrancy is guarded by an internal flag that prevents nested renders during a pass.
- A loop-limit guard stops runaway re-renders within a single cycle and emits a warning.

## Observed attributes

- The host observes changes to `data`, `*methods` and `n-methods`, `*updated` and `n-updated`.
- Other attributes are processed by the binding system during rendering but are not specially parsed on attribute-change alone.

## Transient stores and cleanup

- `$upload`, `$download`, and any keys designated by `*into` are visible for the cycle that produced them and are cleared in the next-frame finalization.
- Clearing happens after `*updated` callbacks for that cycle.

## Configuration switches

- Auto-define - implementations may auto-register the default tag (for example `<serc-rod>`). A configuration flag may disable auto-define so users can register custom tags manually.
- Strict mode - `*strict` enables stricter evaluation rules for its subtree as defined by the implementation.

## Security notes

- `*compose` inserts HTML produced by expressions. The `html` filter is the hook point for sanitization. The default behavior is implementation-defined.
- When binding attributes via expressions, the `attr` filter is responsible for normalization and any required escaping.

## Internal indexes

- Implementations maintain internal indexes for:
  - detecting directives and event handlers on elements
  - tracking transient `*into` keys for finalization
  - correlating data proxies with hosts for scheduling
- Indexes are per host and per world and are not shared across worlds.

---
Back to index: [`README.md`](./README.md)
