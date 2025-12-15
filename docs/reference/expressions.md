# Expressions

_What you will find:_ normative rules for how Sercrod evaluates expressions: where they can appear, what names are in scope, what side effects do, and what is guaranteed. No tutorials or design commentary are included.

## Where expressions appear

Expressions are JavaScript snippets evaluated by the runtime in these contexts:

- Text interpolation: `%expr%`
- Directive values: e.g., `*if="expr"`, `*let="assignments"`, `*print="expr"`
- Attribute shorthands: `:class="expr"`, `:style="expr"`
- Event handlers: `@type="expr"`

Each context may inject additional identifiers (see “Evaluation context”).

## Syntax and semantics

- Expressions are parsed as JavaScript expressions. Statement syntax (e.g., `if (...) { ... }`) is not required to be supported.
- Literals, identifiers, property access, function calls, array/object literals, the comma operator, and conditional operators are valid.
- Return value is the last evaluated value of the expression. Host uses it according to the directive or binding.
- Expressions are evaluated synchronously. Asynchronous work must be performed by directives designed for it (`*fetch`, `*post`, `*api`, `*upload`, `*download`, `*websocket`).

## Evaluation context

Identifiers resolve in this order unless a directive defines stricter rules:

1) **Current scope** - the host’s reactive data object and any variables introduced by `*let`.  
2) **Declared methods** - names exposed via `*methods` and per-world registrations.  
3) **Ambient environment** - normal JavaScript resolution on `globalThis` (e.g., `Math`, `Date`) if not shadowed.

Special identifiers:

- `$event` - only in event handler expressions; the native DOM `Event`.
- `el` - only in event handler expressions; the element that declared the handler.
- `$parent` - only during `*let` evaluation; a writable reference to the nearest ancestor host’s reactive data.
- `$upload`, `$download` - transient stores set by file operations for the cycle that produced them.

Notes:

- `this` is not defined by the runtime for expressions. Do not rely on `this` binding.
- Shadowing follows JavaScript rules; variables from `*let` or data keys can shadow ambient names.

## Side effects and update scheduling

- Reading from the scope has no side effects.
- Writing to reactive properties (e.g., `count++`, `items.push(x)`) schedules a host update according to the scheduler’s coalescing rules.
- Event handlers may use modifiers that affect scheduling:
  - `.noupdate` prevents auto-scheduling for that handler.
  - `.update` forces an update after handler execution.
- Non-input events do not auto-schedule unless they mutate data or use `.update`.

## Result usage by context

- Interpolation `%expr%`: the value is converted to text via filters (`text`, `placeholder`).
- `*print`: the value is converted to text via `text`.
- `*compose`: the value is converted to HTML via `html`.
- `:class`: accepts string, array of strings, or object map `{ name: boolean }`; normalized by `attr`.
- `:style`: accepts CSS string or object map `{ prop: value }`; normalized by `style`.
- `*if`, `*elseif`: the value is tested for truthiness per JavaScript rules.
- `*case`, `*case.break`: the value is compared by strict equality (`===`) to the switch selector.
- Input bindings (`*input`): conversions go through `input_in` and `input_out`.

## `*let` and scope mutation

- `*let="assignments"` evaluates left-to-right, allowing later bindings to reference earlier ones.
- During `*let`, `$parent` is injected and writable for the duration of that evaluation.
- Assignments in `*let` update the current scope for the element’s subtree.

## Methods in expressions

- Methods listed in `*methods` become identifiers in the evaluation context.
- If a global object name is provided (e.g., `"Utils"`), all of its function properties become available by their property names.
- Host-declared method names take precedence over per-world/global registrations with the same identifier.

## Errors and protections

- Exceptions thrown while evaluating an expression do not crash the host by default. The runtime logs a warning or error and skips the effect of the directive that failed.
- A guard prevents re-entrant renders from within the same update pass. A loop-limit aborts runaway cycles and logs a warning.

## Determinism

- Within a single render pass, expression evaluations are deterministic for the same inputs.
- Implementations may cache intermediate results during a pass. Expressions must be side-effect free unless they intentionally mutate reactive data.

## Security notes

- `*compose` uses the `html` filter for HTML output. Sanitization behavior is defined by the implementation. Treat untrusted input accordingly.
- Attribute bindings pass through the `attr` filter, which is responsible for normalization and safe stringification.

## Reserved and transient names

- `$event`: event handlers only.
- `$parent`: `*let` evaluation only.
- `$upload`, `$download`: set by file operations; cleared in the next-frame finalization of the cycle that produced them.
- Keys named by `*into`: transient; cleared in the same finalization phase.

---
Back to index: [`README.md`](./README.md)
