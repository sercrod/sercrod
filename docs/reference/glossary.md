# Glossary

_What you will find:_ concise, alphabetized definitions of terms used in the Sercrod reference. Entries are normative and link to the relevant pages.

## A

**Ambient environment**  
JavaScript global scope available to expressions after scope and methods resolution. See [Expressions](./expressions.md).

**AST (abstract syntax tree)**  
A lightweight structure parsed from the host template and consumed by the renderer. See [Advanced](./advanced.md).

**AST hook**  
A function that receives and may transform the parsed AST before rendering. Registered per world. See [Advanced](./advanced.md).

**Asterisk form**  
Directive spelling that starts with `*` such as `*if`. Equivalent to the `n-` form. See [Directives](./directives.md).

## B

**Binding**  
Association between an expression and a DOM location - text, attributes, events, or control state. See [Directives](./directives.md) and [Expressions](./expressions.md).

## C

**Cleanup (finalization)**  
The deferred step that clears transient stores on the next frame after a render cycle. See [Lifecycle](./lifecycle.md).

**CustomEvents**  
Events dispatched by the host to report lifecycle and side-effect results. See [Events](./events.md).

## D

**Data (host data)**  
Reactive object powering expressions and bindings for a host. Backed by a Proxy. See [Core Concepts](./core-concepts.md).

**Directive**  
An attribute that instructs Sercrod. Exists in `*` and `n-` forms with equivalent behavior. See [Directives](./directives.md).

## E

**Event handler**  
Attribute `@type="expr"` that runs an expression on native events. Supports modifiers such as `.prevent`, `.stop`. See [Directives](./directives.md).

**Expression**  
JavaScript snippet evaluated by Sercrod in a defined context. See [Expressions](./expressions.md).

## F

**Filter**  
Named transformation applied at fixed points such as text, HTML, attributes, style, and input conversions. See [Filters](./filters.md).

## H

**Host (Sercrod host)**  
Custom element instance that runs Sercrod for its subtree. Defaults to `<serc-rod>`. See [Host API](./host-api.md).

## I

**Into key**  
Property name designated by `*into="name"` to store results of HTTP or file operations for the current cycle. Cleared in finalization. See [Communication](./communication.md).

**Interpolation**  
Inline text expansion using `%expr%`. See [Expressions](./expressions.md).

## L

**Lifecycle**  
Ordered phases from construction and connection to rendering and finalization. See [Lifecycle](./lifecycle.md).

## M

**Methods**  
Functions exposed to expressions via `*methods` or global registrations. Resolved by a fixed lookup order. See [Host API](./host-api.md) and [Expressions](./expressions.md).

## N

**`n-` form**  
Directive spelling using `n-` such as `n-if`. Equivalent to the asterisk form. See [Directives](./directives.md).

## O

**Observed attributes**  
Subset of attributes whose changes trigger special parsing and an update: `data`, `*methods` or `n-methods`, `*updated` or `n-updated`. See [Host API](./host-api.md).

## P

**Pre-hook**  
Function that mutates parsed DOM before AST extraction. When any pre-hook is present, template string caching is bypassed. See [Advanced](./advanced.md).

**Proxy wrapping**  
Mechanism that wraps the host data object so writes can schedule updates. See [Core Concepts](./core-concepts.md).

## R

**Render pass**  
A single execution of the update pipeline that evaluates bindings, applies side effects, commits DOM, and schedules finalization. See [Lifecycle](./lifecycle.md).

## S

**Scheduler**  
Frame-based coalescing that batches multiple triggers into one render per host per frame. See [Lifecycle](./lifecycle.md).

**Scope**  
The effective namespace for expression evaluation - primarily the host data plus `*let` locals and declared methods. See [Expressions](./expressions.md).

**Shorthand attributes**  
`:class` and `:style` convenience bindings that map to attribute updates via filters. See [Directives](./directives.md) and [Filters](./filters.md).

**Stage buffer**  
Optional working copy enabled by `*stage`. Writes flow to the buffer until `*apply` or `*restore`. See [Directives](./directives.md).

**Strict mode**  
Stricter evaluation rules for a subtree activated by `*strict` as supported by the implementation. See [Advanced](./advanced.md) and [Configuration](./configuration.md).

## T

**Template**  
String captured from the host’s initial `innerHTML` that serves as the source for AST extraction and rendering. See [Advanced](./advanced.md).

**Template cache**  
Map keyed by template string that stores parsed ASTs. Disabled when pre-hooks are active. See [Advanced](./advanced.md).

**Transient store**  
Short-lived data fields cleared in finalization. Includes `$upload`, `$download`, and all keys recorded by `*into`. See [Lifecycle](./lifecycle.md) and [Communication](./communication.md).

## U

**Update cycle**  
A scheduled render pass triggered by reactive data writes, observed-attribute changes, event handlers, or side-effect completions. See [Lifecycle](./lifecycle.md).

## W

**Warnings and errors**  
Reporting policy for failures and guard trips. See [Warnings and Errors](./warnings-and-errors.md).

**WebSocket helper**  
Helper object exposed on the host when `*websocket` is active, providing `connect`, `reconnect`, `close`, `send`, and `status`. See [WebSocket](./websocket.md).

**World**  
Isolation namespace that scopes registries and caches for a set of hosts sharing a registration. See [World](./world.md).
