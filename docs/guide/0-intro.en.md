# Guide 0 - start here

Last updated: 2025-10-25  
Status: draft

## What you will do in 30 seconds
- Load `sercrod.js`.
- Add one directive in HTML (`*let`) and print with `%...%`.
- See an immediate update - no build step required.

## Why Sercrod (one line)
Attribute-first and tiny. The browser already handles attributes -> data; Sercrod adds a small, explicit path for data -> attributes when you need it.

## Try it now

### Static hero (3 lines)
```html
<serc-rod>
  <p *let="message='Hello, Sercrod'">%message%</p>
</serc-rod>
```

### Tiny dynamic (4 lines)
```html
<serc-rod data='{"name":"world"}'>
  <p>Hello, %name%!</p>
  <input type="text" *input="name">
</serc-rod>
```

Prefer files? See `./snippets/intro-hero.html` and `./snippets/intro-dynamic.html`.

## Learn the rest (roadmap 1 - 8)
1. **Getting started** - how to load the script and verify installation.  
2. **Directives overview** - what each directive does at a glance.  
3. **Data fundamentals** - scope, `*let`, and how data flows.  
4. **Binding & events (basics)** - `%...%` interpolation, event handling (for example, `@click`), `*input`.  
5. **Lifecycle - load & save (basics)** - working with local files and events.  
6. **Lifecycle - fetch & post (advanced)** - server roundtrip basics.  
7. **Binding & events (advanced)** - deeper patterns and edge cases.  
8. **Nested Sercrod** - composing multiple Sercrod hosts.

## Quick glossary (lite)
- directive - attribute mini-commands like `*each`, `*print`, `*let`, `*input`.  
- interpolation - inline expansion with `%name%` (example: `%message%`).  
- watch mode - reactivity scope: `off` / `observed` / `all` / `super`.

## Notes
Sercrod is the project name used in this guide. For background and comparisons, see the Reference Intro.

## Legal
"Sercrod" and the Sercrod logo are trademarks or pending trademarks of Hitoshi Watase. Use must follow our brand guidelines [link-to-brand-guidelines]. Other names are the property of their owners.
