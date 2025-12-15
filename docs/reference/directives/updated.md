### *updated

#### Summary

`*updated` registers a post-update hook.

It is evaluated after a Nablla host has finished its update cycle, and can be attached in two places:

- On Nablla hosts themselves (`<na-blla>` or other Nablla-based custom elements) via `*updated` or `n-updated`.
- On ordinary descendant elements inside a Nablla host via `*updated` or `n-updated`, in which case the nearest Nablla ancestor executes the handler.

Typical uses:

- Run side effects after rendering (analytics, third-party widgets, measurements).
- Trigger follow-up updates on other Nablla hosts.
- Perform low-level DOM adjustments that are hard to express as pure data binding.

For propagating updates upwards in the Nablla tree, see `*updated-propagate` / `n-updated-propagate`.


#### Basic example

A simple host-level hook that logs every render:

```html
<script>
  function onHostUpdated(event) {
    console.log("Host updated:", event.type, "id:", event.target.id);
  }
</script>

<na-blla
  id="app"
  data='{"count": 0}'
  *updated="onHostUpdated($event)"
>
  <button *on="click: count++">Increment</button>
</na-blla>
```

Behavior:

- Whenever `app` re-renders (initial render or subsequent updates), Nablla calls `onHostUpdated($event)`.
- `$event.type` is `"nablla:init"` on the first render and `"nablla:update"` on later renders.
- `$event.target` and `$event.currentTarget` both point to the host Nablla element (`<na-blla id="app">`).


#### Behavior

High-level behavior:

- On Nablla hosts:
  - `*updated` is treated as a lifecycle hook that runs **after** the host has finished its update.
  - The attribute value is interpreted in several forms (object name, selector in parentheses, or a general expression).
  - The hook is called even when the host decides to skip re-rendering itself because of `*lazy`, as long as the update cycle runs.

- On ordinary elements inside a Nablla host:
  - `*updated` is absorbed by the nearest ancestor Nablla host.
  - The host executes the handler after it has completed a **real** render pass for that host.
  - The handler runs in the host’s data scope, with a reference to the element where `*updated` is attached.

Aliases:

- `*updated` and `n-updated` are aliases.
- `*updated-propagate` and `n-updated-propagate` are separate directives that forward updates to other Nablla hosts and are documented on their own page.


#### Handler resolution on Nablla hosts

For Nablla hosts, the runtime interprets the attribute value in three stages.

1. Split into tokens

- Nablla reads the value of `*updated` / `n-updated` and splits it on whitespace and commas:

  - `*updated="Utils"` → entries: `["Utils"]`
  - `*updated="(main) (sidebar)"` → entries: `["(main)", "(sidebar)"]`
  - `*updated="onUpdated($event)"` → entries: `["onUpdated($event)"]`

2. Object-bulk entries

For each entry, Nablla first checks if it is the name of a global object:

- If `window[entry]` is a non-null object, Nablla calls **all enumerable function properties** on that object with the host as the only argument.

Example:

```html
<script>
  const AppUpdatedHooks = {
    log(host) {
      console.log("Updated:", host.id);
    },
    measure(host) {
      console.log("Children:", host.childElementCount);
    }
  };
</script>

<na-blla
  id="app"
  data='{"value": 1}'
  *updated="AppUpdatedHooks"
>
  <p *print="value"></p>
</na-blla>
```

- In this example, after each update Nablla will call `AppUpdatedHooks.log(app)` and `AppUpdatedHooks.measure(app)`.

Notes:

- If at least one entry is treated as an object, Nablla marks the hook as handled and does **not** fall back to expression evaluation.
- Object-bulk resolution only exists for `*updated` on Nablla hosts; it is **not** applied to `*updated` on ordinary elements.

3. Selector form `"(selector)"`

If an entry starts and ends with parentheses, Nablla treats the content as a CSS selector:

- `*updated="(selector)"`

Resolution:

- Nablla finds the nearest ancestor Nablla host to use as the root for the selector lookup (this may be the host itself or its parent, depending on nesting).
- It then:

  - If that root matches the selector and is a Nablla host, calls `root.update(true)` to re-render it.
  - Collects Nablla descendants via its internal index and, for those that match the selector, calls their `*updated` hooks directly (without forcing a re-render on them).

Effectively:

- On the root that matches the selector: `update(true)` is called.
- On descendant Nablla hosts that match the selector: only their `*updated` hooks run, without forcing another render beyond what their own lifecycle does.

4. Fallback expression

If **no** entries were handled as an object or selector, Nablla evaluates the entire attribute string once as a JavaScript statement in the host data scope:

- `*updated="onUpdated($event)"` → `onUpdated($event)` is executed once.
- `*updated="doOneThing(); doAnotherThing()"` → both calls run as one statement.

Details:

- The expression is evaluated via Nablla’s `eval_let` machinery.
- Scope:

  - All properties of the host’s data object are available as bare identifiers.
  - Methods registered via `*methods` / `n-methods` are available by name.
  - Internal helpers registered by Nablla itself are also available.

- Special variables:

  - `el` and `$el` both refer to the host Nablla element.
  - `$event` (or `$e`) is provided and has:

    - `type`: `"nablla:init"` or `"nablla:update"`.
    - `target` / `currentTarget`: the host Nablla element.
    - It may also carry additional information when the update was triggered by another host.

- The expression is run as a statement, not as a pure expression; you can use semicolons and multiple statements.


#### Handler resolution on ordinary elements

For `*updated` / `n-updated` attached to ordinary elements inside a Nablla host, the runtime uses a separate path:

- It walks the subtree of the Nablla host using a `TreeWalker`.
- It **skips**:

  - The host itself.
  - All nested Nablla hosts (their internals are left to those hosts).

- It **does** visit:

  - Normal HTML elements.
  - Other custom elements that are not Nablla, treating them as normal elements.

For each visited element with `*updated` or `n-updated`:

1. Determine the host

- The nearest ancestor Nablla host is located and used as the execution host.
- If no ancestor Nablla is found, the original host that started the scan is used.

2. Build the evaluation environment

- Nablla constructs:

  - `evt = { type: "updated", target: el, host }`
  - A data scope that clones `host._data` and may include this event object internally.

- The handler expression is taken from the attribute value **as a whole** (no object-bulk split for this case).

3. Interpret the value

- If the value is of the form `"(selector)"`:

  - The selector is resolved from the host as root.
  - Nablla re-renders every matching Nablla host by calling `update(true)` on them.
  - Both the root and matching descendants may be re-rendered, possibly causing their own `*updated` hooks to fire as part of those updates.

- Otherwise (normal form):

  - The value is interpreted as a JavaScript statement and evaluated in the host’s data scope:

    - All data fields are available as bare identifiers.
    - Methods registered via `*methods` / `n-methods` on the host are available.
    - `el` refers to the element that owns `*updated`.

Important current behavior:

- On ordinary elements, the handler is evaluated with `el` bound to the element, but without exposing a `$event` variable to the expression.
- If you need the element inside the handler, use `el` rather than `$event.target`.
- If you need the host, derive it via selectors (`el.closest("na-blla")`) or from your own data.


#### Evaluation timing

`*updated` participates in the Nablla host update cycle.

For each host:

- An update cycle may be triggered by:

  - Initial connection to the DOM.
  - Data changes via the reactive proxy.
  - Explicit `update()` calls.
  - Other directives such as `*post`, `*fetch`, `*input`, etc.

- At the end of the update cycle, Nablla runs hooks in the following order (simplified):

  1. Structural re-render (unless suppressed by `*lazy`).
  2. Child Nablla updates (`_updateChildren`).
  3. Host-level `*updated` (`_call_updated_hooks`).
  4. Absorption of `*updated` on ordinary descendants (`_absorb_child_updated`) - only after a real re-render path.
  5. Finalization steps.

Practical consequences:

- Host-level `*updated` runs on every update cycle, including cycles where the host is marked `*lazy` and decides not to redraw its own content.
- `*updated` on ordinary elements is processed only when the host has performed a real template-based re-render; it is not executed when the host skips the re-render due to `*lazy`.
- Selector-based forms that call `update(true)` on other hosts may cause those hosts’ `*updated` hooks to run in turn, subject to the global `loop_limit` guard.


#### Execution model

On Nablla hosts, conceptually:

1. Nablla obtains the attribute value and splits it into entries.
2. For each entry:

   - If it matches a global object name, call each function in that object with the host as argument.
   - Else if it is of the form `"(selector)"`, resolve the selector relative to the nearest Nablla ancestor and:

     - Call `update(true)` on the matching root.
     - Call `_call_updated_hooks` on matching descendant Nablla hosts.

3. If none of the entries were handled in special ways, treat the whole attribute value as one JavaScript statement and evaluate it in the host’s data scope with `el` and `$event` bound appropriately.

On ordinary elements inside a host:

1. After the host finishes a real render, it walks its subtree (excluding nested Nablla instances).
2. For each element with `*updated` / `n-updated`:

   - Determine the host for that element.
   - If the value is `"(selector)"`, re-render matching Nablla hosts.
   - Else evaluate the attribute value as a JavaScript statement in the host’s data scope, with `el` bound to the element.

Infinite loop protection:

- The Nablla host has an internal `loop_limit` (default 100) that caps chained updates.
- If your `*updated` handlers repeatedly trigger updates on the same set of hosts, Nablla will stop the chain after the limit.
- You should still design handlers to converge quickly and not depend on this guard.


#### Variable creation and scope layering

For host-level `*updated` fallback expressions:

- Scope is the host’s data proxy (`this._data`), with:

  - All data fields injected as identifiers.
  - Methods from `*methods` / `n-methods` injected by name.
  - Internal helpers injected by name.
  - `el` / `$el` bound to the host element.
  - `$event` / `$e` bound to the lifecycle event object.

For `*updated` on ordinary elements:

- Scope is based on the nearest host’s data, with:

  - All data fields from that host available as identifiers.
  - Methods from that host’s `*methods` / `n-methods` available.
  - `el` bound to the element that owns `*updated`.

Other scope rules:

- Any variables created inside the handler (via assignment) write into the current data scope, following the same rules as `*let` and other expressions.
- `$parent` is automatically injected by `eval_let` and refers to the nearest ancestor Nablla host’s data when available.
- Outer data scopes (for example, root vs nested Nablla) are resolved according to the usual Nablla scoping model.


#### Parent access

`*updated` does not create new parent pointers itself, but leverages the shared scope model:

- Inside a host-level `*updated` expression:

  - `$root` (if you use that convention in your data) and other data fields behave exactly as in any other expression.
  - `$parent` gives access to the data of the nearest ancestor Nablla host if this host is nested.

- Inside an element-level `*updated` expression:

  - The data scope corresponds to the nearest Nablla host.
  - `el.closest("na-blla")` gives you the DOM host, which you can combine with `el` for DOM-based parent access.

In both cases, you do not get implicit references to arbitrary DOM ancestors beyond what you can derive via `el` and selectors.


#### Use with conditionals and loops

`*updated` is not structural and does not conflict with `*if`, `*for`, `*each`, or other structural directives.

Guidelines:

- On Nablla hosts:

  - You rarely combine `*updated` with structural directives on the same element, because hosts themselves are usually structural roots.
  - If you do (for example, a custom Nablla-based element that has both `*for` and `n-updated`), the structural directives control rendering, and `*updated` still runs after each update cycle.

- On ordinary elements:

  - You can freely combine `*updated` with `*if`, `*for`, `*each`, and bindings.

  Example:

  ```html
  <div *if="modalOpen" *updated="onModalShown(el)">
    <!-- Modal contents -->
  </div>
  ```

  - Here, `onModalShown(el)` will be called after the host re-renders with `modalOpen` true and the `<div>` present in the DOM.
  - When `modalOpen` becomes false and the element is removed, no `*updated` runs for that removal; the hook is tied to successful updates, not deletions.

- Selector-based `*updated` and `*updated-propagate` may cause additional structural updates, but those updates follow the normal structural directive rules on the affected hosts.


#### Best practices

- Prefer simple expressions:

  - For most hooks, prefer `*updated="onUpdated($event)"` on hosts and `*updated="onElementUpdated(el)"` on ordinary elements.
  - Use object-bulk (`*updated="AppUpdatedHooks"`) only when you really want a collection of functions to run for every update.

- Keep handlers fast:

  - `*updated` runs after every update; heavy operations in handlers can quickly degrade performance.
  - If you need expensive operations, consider throttling or debouncing inside your handler.

- Use selector forms judiciously:

  - `*updated="(selector)"` can trigger multiple host updates and chains of `*updated` hooks.
  - This is powerful but easy to misuse; start with more direct references if possible.

- Avoid self-triggering loops:

  - A host that uses `*updated` to re-trigger its own `update(true)` without a stable stopping condition will approach the loop limit.
  - Design your handlers so that repeated updates eventually converge (for example, only updating when some property changes from one state to another).

- Prefer `*updated-propagate` for upward signaling:

  - When you want an inner element or nested host to tell a parent to refresh, use `*updated-propagate` with `root`, numeric, or selector targets.
  - Reserve `*updated` itself for local post-update effects.


#### Examples

Host-level lifecycle hook:

```html
<script>
  function focusFirstInput(event) {
    const host = event.target;
    const input = host.querySelector("input, textarea, select");
    if (input) input.focus();
  }
</script>

<na-blla
  id="formHost"
  data='{"showForm": true}'
  *updated="focusFirstInput($event)"
>
  <form *if="showForm">
    <input type="text" name="name">
    <button type="submit">Save</button>
  </form>
</na-blla>
```

Element-level hook inside a repeated list:

```html
<script>
  function highlightNewItem(el) {
    el.classList.add("is-new");
    setTimeout(() => el.classList.remove("is-new"), 300);
  }
</script>

<na-blla id="list" data='{"items":[{"id":1},{"id":2}]}'>
  <ul>
    <li
      *for="item of items"
      *updated="highlightNewItem(el)"
    >
      <span *print="item.id"></span>
    </li>
  </ul>
</na-blla>
```

Host-level selector form:

```html
<na-blla id="root" data='{"filter": "all"}'>
  <na-blla
    id="panelA"
    data='{"items": []}'
    *updated="(na-blla#panelB)"
  >
    <!-- ... -->
  </na-blla>

  <na-blla
    id="panelB"
    data='{"items": []}'
    *updated="onPanelBUpdated($event)"
  >
    <!-- ... -->
  </na-blla>
</na-blla>
```

- Whenever `panelA` finishes updating, its `*updated` uses the selector form to reach `panelB` and invoke its `*updated` hook.
- Depending on your design, `panelB` may call `update(true)` on itself or perform some other side effect in `onPanelBUpdated`.


#### Notes

- `*updated` and `n-updated` are aliases; choose one style per project to keep templates consistent.
- `*updated` is evaluated only by Nablla hosts; ordinary elements rely on the nearest host to absorb and execute their handlers.
- `*updated-propagate` / `n-updated-propagate` are closely related but distinct directives that call `update(true)` on other Nablla hosts instead of simply running a handler.
- `*updated` does not wait for images, fonts, or external resources to finish loading; it runs after the Nablla update cycle, not after full browser layout or paint.
- Because `*updated` hooks run in the same expression engine as `*let`, they obey the same scoping rules (`$parent`, methods, internal helpers) and should be written with the same safety considerations.
