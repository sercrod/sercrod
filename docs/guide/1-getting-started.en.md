# Getting Started with Sercrod
_Last updated: 2025-10-22_

> This chapter helps you set up Sercrod and understand its basic structure through short examples.  

## 1) Setup
Sercrod works in any modern browser ? no build tools, no bundlers.

<!-- Include Sercrod from CDN -->
```html
<script src="https://cdn.jsdelivr.net/npm/sercrod@1.0.0/dist/sercrod.min.js"></script>
```

<!-- Or include a local build -->
```html
<script src="./index.js"></script>
```

Once loaded, Sercrod registers the `<serc-rod>` element and enables attribute directives.

---

## 2) First Example
A minimal Sercrod instance written as pure HTML.

<!-- Minimal one-liner -->
```html
<serc-rod>
  <p *let="message='Hello Sercrod!'">%message%</p>
</serc-rod>
```

What it shows: define a local value with `*let`, render it with `%message%`. No JavaScript required.

---

## 3) Add Some Interaction
Make the message editable by the user.

<!-- Interactive sample -->
```html
<serc-rod data='{"name":"World"}'>
    <h2>Hello, %name%!</h2>
    <input *input="name" placeholder="Type your name..." />
</serc-rod>
```

Type in the input and watch the text update. `*input` keeps `name` in sync both ways.

---

## 4) How It Works
| Flow | Role |
|------|------|
| attributes -> data | handled by Web Components (`attributeChangedCallback`) |
| data -> attributes | handled by Sercrod (writes back only when needed) |

No virtual DOM. No custom runtime. HTML remains the source of truth.

---

## 5) Snippets (external files)
- Snippet #1: `./snippets/intro-hero.html`
- Snippet #2: `./snippets/intro-dynamic.html`

---

## 6) Next Steps
- Learn more directives: `*let`, `*text`, `*if`, `*input`, etc.
- Read the philosophy in `docs/0-intro.en.md`.
- Contribute at: https://github.com/hitoshi-watase/sercrod

---

(c) 2025 Hitoshi Watase - Sercrod Project.  
License: GNU General Public License v3.0 or later (GPLv3+)
