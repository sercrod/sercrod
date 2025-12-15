# World

_What you will find:_ normative rules for world isolation. This page defines how hosts are grouped by world, how registries are separated, and how cross-world interactions behave. No design commentary is included.

## Definitions

- **Host**: a custom element instance that runs Sercrod.
- **World**: a logical namespace that groups hosts sharing the same Sercrod registration. Worlds own their registries and caches.
- **Global layer**: implementation-wide defaults that may be visible to all worlds unless overridden.

## World creation and naming

- A world is created when a Sercrod custom element is defined.
- Each tag name bound to the Sercrod implementation belongs to one world. Implementations may derive the world key from the tag name.
- Multiple distinct tags may map to the same Sercrod constructor. If so, they are part of the same world unless the implementation chooses to separate them.

## Membership and boundaries

- Every host belongs to exactly one world. Nested hosts can be in the same world or in different worlds.
- Worlds are isolated:
  - Registries, caches, and reactive maps are not shared across worlds.
  - Updates, schedulers, and lifecycle state are per host and do not propagate across worlds unless data sharing is explicitly configured by the user.

## Registries and lookup order

Worlds maintain their own registries. The effective lookup order is:

1. **World registry** - entries registered in the current world.
2. **Global layer** - implementation defaults visible to all worlds.
3. **Ambient JavaScript** - normal name resolution on `window` or `globalThis` where applicable.

The following components follow this lookup order:

- **Methods exposure** used by expressions.
- **Filters** applied to text, attributes, style, and input conversions.
- **AST hooks** that run after template parse.
- **Internal reactive maps** that track proxies and raw objects.

Implementations may expose registration functions that either target the current world or the global layer. The exact API surface is defined by the runtime.

## `$parent` across worlds

- During `*let` evaluation, `$parent` refers to the nearest ancestor host that the implementation recognizes as a Sercrod host.
- If the ancestor belongs to a different world that is not recognized as the same implementation, `$parent` is not linked.
- If multiple tags are registered to the same Sercrod implementation, `$parent` resolution can traverse them as a single lineage.

## Scheduler independence

- Each host schedules its own renders. Coalescing and loop guards are per host.
- A child host updating in another world does not trigger the parent to re-render unless shared data is mutated or an explicit signal is used.

## Caches and invalidation

- Template parse caches and AST caches are world-scoped.
- Invalidations performed in one world do not affect caches in another world.

## Cross-world communication

- Worlds do not share data or registries implicitly.
- Cross-world interaction, if needed, must use explicit channels such as:
  - Writing to shared ambient objects on `window` or `globalThis`.
  - Posting messages or dispatching CustomEvents and listening from the other host.
  - Using network or storage mechanisms that both worlds can observe.
- `*global` writes only affect the current host data if a same-named key exists, otherwise they write to the ambient environment. This does not change world registries.

## Errors and protections

- Registration collisions are resolved within the current world first. Implementations may warn on conflicts in the same world.
- Failures in hooks or filters are contained to the world where they occur and do not impact other worlds.

---
Next page: [`advanced.md`](./advanced.md)
