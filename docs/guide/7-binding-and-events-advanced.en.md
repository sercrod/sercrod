# Binding & Events (advanced)
_Last updated: 2025-10-22_

>This chapter explains how Sercrod exchanges data with a server, extending the local lifecycle introduced in the Chapter 4: Binding & Events.  

---

## 1. Overview  

This chapter extends the binding concepts introduced earlier.  
It covers additional output and attribute bindings,  
showing how Sercrod continues to rely on the same data flow model.  

---

## 2. *compose: HTML output  

`*compose` writes HTML instead of plain text.  
It interprets tags inside the value and inserts them as real elements.  

```html
<serc-rod data='{"content":"A <i>quick</i> brown fox jumps over the <b>lazy</b> dog."}'>
  <p *compose="content"></p>
</serc-rod>

<p>A <i>quick</i> brown fox jumps over the <b>lazy</b> dog.</p>  
^ Rendered output (omit `<serc-rod>`)  

```
Use `*compose` only when you intend to include HTML elements inside data.  
For ordinary text, `*print` is preferred.  

---

## 3. Attribute bindings (:prop syntax)  

Sercrod can also bind data to element attributes using the colon (`:`) prefix.  
This updates the attribute whenever the bound value changes.  

```html
<serc-rod data='{"link":"https://example.com","label":"Open link"}'>
  <a :href="link" *print="label"></a>
</serc-rod>

<a href="https://example.com">Open link</a>  
^ Rendered output (omit `<serc-rod>`)  

```

The colon syntax connects the attribute to the same reactive data used by `*print` and `*compose`.  
When the value of `link` changes, the `href` attribute updates automatically.  

---

This chapter adds minor but important extensions to the binding model introduced in Chapter 4.
The same data flow applies ? only the output methods differ.

---

## 4. Summary  

Directive | Purpose | Handles HTML | Typical use  
-----------|----------|---------------|--------------  
`*print` | Write text only | No | Display plain values  
`*compose` | Write HTML content | Yes | Insert markup or formatted text  
`:prop` | Bind attribute value | N/A | Update attributes dynamically  

Each directive has its own role.  
Choose according to whether you need plain text, markup, or attribute updates.
