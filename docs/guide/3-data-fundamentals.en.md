# Data Fundamentals
_Last updated: 2025-10-22_

> This chapter explains how data is defined, stored, and displayed in Sercrod, using simple examples with *let and data attributes.  

## 3) From `*let` to `data` - the smallest ladder

#### Guidance - Prefer `data` over `*let`

In Chapter 2, we used `*let` to keep examples minimal and easy to follow.  
In everyday code, define values in `data` and read them with directives (`*print`, `:prop`, etc.).  
Use *let only for short-lived locals (loops/branches).

### Step 1: Recap (`*let` makes a value and shows it)

```html
<serc-rod>
  <p *let="message = 'Hello Sercrod!'" *print="message"></p>
</serc-rod>

<p>Hello Sercrod!</p>
^ rendered output (omit `<serc-rod>`)

```

### Step 2: Make it stable (move the same value into `data`)

```html
<serc-rod data='{"message":"Hello Sercrod!"}'>
  <p *print="message"></p>
</serc-rod>

<p>Hello Sercrod!</p>
^ rendered output (omit `<serc-rod>`)
```

### Step 3: Another primitive (number)

```html
<serc-rod data='{"count":0}'>
  <p *print="count"></p>
</serc-rod>

<p>0</p>
^ Rendered output (omit `<serc-rod>`)
```

### Step 4: Quick variants (boolean and null)

```html
<serc-rod data='{"flag":true}'>
  <p *print="flag"></p>
</serc-rod>

<p>true</p>
^ Rendered output (omit `<serc-rod>`)
```

```html
<serc-rod data='{"note":null}'>
  <p *print="note"></p>
</serc-rod>

<p></p>
^ Rendered output (omit `<serc-rod>`)
```

### Step 5: A tiny object (two fields)

```html
<serc-rod data='{"item":"Pencil","stock":12}'>
  <p *print="item"></p>
  <p *print="stock"></p>
</serc-rod>

<p>Pencil</p>
<p>12</p>
^ Rendered output (omit `<serc-rod>`)
```

### Step 6: A small array (iterate with `*for`)

```html
<serc-rod data='{"items":["Pencil","Notebook","Eraser"]}'>
  <ul>
    <li *for="item of items">%item%</li>
  </ul>
</serc-rod>

<ul>
  <li>Pencil</li>
  <li>Notebook</li>
  <li>Eraser</li>
</ul>
^ Rendered output (omit `<serc-rod>`)
```

### Note: `data` is always text

The `data` attribute is always a text value. Sercrod reads and parses it as JSON when the component starts.  
This is why all keys and strings must be quoted and why functions cannot be placed directly inside `data`.

## Summary

1. **Data stays simple** - start with one value, then small objects or arrays.  
2. **Prefer `data` over `*let`** - use `data` for anything persistent or shared.  
3. **Keep JSON valid and small** - double quotes, flat shape, no functions.  
4. **Arrays loop naturally with `*for`** - no helpers needed.
