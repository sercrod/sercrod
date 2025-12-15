# Introduction (Why)
_Last updated: 2025-10-14_

## 1) Why this exists
Templates and “diff layers” tend to grow. They slow down learning, debugging, and rendering.  
**Sercrod** treats **HTML attributes as the source of truth**. The browser already tells us when attributes change (`observedAttributes`, `attributeChangedCallback`).  
Sercrod adds the missing half: a small, explicit path **data -> attributes**.

## 2) What’s different
- **No virtual DOM.** No custom runtime layer.  
- **No MutationObserver.** Attribute change detection comes from Web Components.  
- **Works with SSR, static HTML, and CSR.**  
- **Clear responsibility:** attributes -> data (browser), data -> attributes (Sercrod).

## 3) Comparisons
- **Small page:** add a few directives and go. No build step required for the basics.  
- **Mid-size app:** fewer update paths to trace; attributes are the forward path; `data -> attributes` only when needed.  
- **Large DOM:** scope reactivity with watch modes: `off / observed / all / super`.

## 4) Summary
**Attribute-first.** Minimal write-back. Readable code; updates are explicit and traceable.

---

## In one line
Sercrod is a tiny, attribute-first layer that writes **data -> attributes** only when needed.  
It pairs with Web Components’ built-in **attributes -> data** flow, so your stack stays small and predictable.

## 30-second sample (hero)
Single tag. No extra JS. A local value via `*let`, printed by `%…%`.  
Snippet #1: [./snippets/intro-hero.html](./snippets/intro-hero.html)

## Optional: tiny dynamic sample (feel the update)
A minimal “type-to-update” feel.  
Snippet #2: [./snippets/intro-dynamic.html](./snippets/intro-dynamic.html)

## Quick glossary
- **directive:** attribute mini-commands like `*each`, `*text`, `*let`, `*model`.  
- **interpolation:** inline expansion with `%name%` (e.g., `%message%`).  
- **watch mode:** reactivity scope ? `off` (none) / `observed` (only marked parts) / `all` (shallow all) / `super` (global).  
- **diff layers:** abstractions for change detection/patching. Sercrod keeps this minimal.

## Getting started (3 steps)
1) **Load** `sercrod.js` (from your build or a CDN).  
2) **Write** directives in HTML (e.g., `*each`, `*text`, `*let`, `*model`).  
3) **Provide data** to your host element: `element.data = {...}` and update it as state changes.  
   Sercrod writes back to attributes; Web Components handle the rest.

We use **Sercrod** as the project name here. Chapters follow: Why -> What’s different -> Comparisons -> Summary.

---

### Legal / Trademark

"Sercrod" and the Sercrod logo are trademarks or pending trademarks of Hitoshi Watase. Use of the name and logo must follow our brand guidelines ([link-to-brand-guidelines]). All other product names and marks are the property of their respective owners.

This documentation is provided for technical information only and does not constitute legal advice.

