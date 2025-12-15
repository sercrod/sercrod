# Core Concepts

_What you will find:_ the data model, scope rules, expression evaluation, interpolation, and how Sercrod treats attributes and methods. This page defines the mental model used throughout the reference.

## Host and data

- **Host element**: a custom element instance that runs Sercrod. The default tag is `<serc-rod>`.
- **Data source**: the `data` attribute holds a JSON object. At runtime you can also assign `host.data = {...}`.
- **Reactive wrapping**: Sercrod wraps the data object using a Proxy so reads and writes can schedule updates.
- **Immutability policy**: objects are not frozen. You may mutate nested structures; Sercrod will observe and coalesce updates per frame.

## Scope

- **Current scope**: the scope for expressions is the host’s reactive data object.
- **Parent scope**: within `*let` evaluation, Sercrod injects `$parent` that points to the nearest ancestor host’s data. `$parent` is read-write and refers to the parent host’s reactive object.
- **Methods presence**: names registered through `*methods` become identifiers in the scope during evaluation. See Methods below.

## Expressions

- **Form**: expressions are JavaScript snippets evaluated in the current scope.
- **Where they appear**:
  - Inside text interpolation: `%expr%`
  - In directive values: e.g. `*if="count > 0"`
  - In attribute shorthands: e.g. `:class="expr"`, `:style="expr"`
  - In event handlers: `@click="expr"`
- **Evaluation environment**:
  - The scope object is the primary namespace.
  - Registered methods are injected as identifiers.
  - Global objects remain accessible through the normal JavaScript name resolution.
  - In event handlers, `$event` is the native DOM Event. A local `el` refers to the element that attached the handler.
  - Outside event handlers, `el` is not injected.

## Interpolation and output

- **Text interpolation**: `%expr%` inserts the evaluated result into a text node.
- **Output directives**:
  - `*print` writes plain text.
  - `*compose` writes HTML.
  - `*textContent` and `*innerHTML` are synonyms for the above output behaviors.
- **Filters**: Sercrod applies built-in filters at well-defined points:
  - `text`, `html`, and `placeholder` for text interpolation and content output.
  - `attr` for attribute values.
  - `style` for style objects or strings.
  - `input_in` and `input_out` for form element value conversion.

## Attributes and shorthands

- **Dynamic attributes**: any attribute can be bound by using an expression as its value through Sercrod’s binding rules.
- **Shorthands**:
  - `:class="expr"` updates `class` based on `expr` (string, array, or object map).
  - `:style="expr"` updates `style` based on `expr` (string or object map).
- **Equivalents**: `n-class` and `n-style` are equivalent to the shorthand forms.

## Input and staging

- **`*input`**: binds a form control to a data key. Direction is two-way through `input_in` and `input_out` filters.
- **Staging buffer**:
  - `*stage` enables a working copy that isolates edits from the main data.
  - `*apply` copies staged values into the main data.
  - `*restore` discards staged changes.
- **Save and load**:
  - `*save` serializes the current data to a downloadable JSON file.
  - `*load` accepts a chosen JSON file and restores values into the host.

## Methods

- **Purpose**: make functions available to expressions without polluting the data object.
- **Declaration**:
  - `*methods="nameA, nameB"` exposes global functions `nameA`, `nameB`.
  - `*methods="SomeObject"` exposes each function property of `window.SomeObject`.
  - The `n-` prefixed form is equivalent.
- **Lookup order**: methods declared on the host, then any globally registered method sets, then normal JavaScript resolution.

## Update model primer

- **Coalescing**: Sercrod batches multiple data changes in the same frame and renders once.
- **Template**: the host captures its initial innerHTML as the template for rendering.
- **AST and hooks**: Sercrod parses templates into a lightweight AST and runs registered AST hooks before rendering. Details are in the Advanced page.

## Reserved names and specials

- `$event`: the DOM event object, only in event handler expressions.
- `$parent`: the nearest ancestor host’s data during `*let` evaluation.
- `$upload` / `$download`: transient stores set by file operations. They are cleared automatically after the update cycle that produced them.
- `*into`: designates a property name where results of network or file operations are stored for the current cycle. Transient keys created through `*into` are also cleared after finalization.

## Equivalence of directive forms

Every directive has two equivalent forms:
- Asterisk form: `*if`, `*each`, `*let`, etc.
- `n-` form: `n-if`, `n-each`, `n-let`, etc.

This document uses the asterisk form for brevity. The `n-` form behaves the same.

---
Next page: [`directives.md`](./directives.md)
