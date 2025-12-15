# Directives Overview  
_Last updated: 2025-10-22_

> This chapter introduces Sercrod’s directive system and explains how HTML attributes control logic and data flow.  

## 1) What Are Directives
Directives are Sercrod’s attribute-based mini-commands.
They extend plain HTML with logic such as loops and conditions ? while remaining valid HTML.
Every directive starts with an asterisk (`*`) and acts on its host element (some affect its children as noted).

Example:
```html
<p *let="message='Hello world!'">%message%</p>

<p>Hello world!</p>
^ rendered output
```

Here, *let defines a local variable message, and %message% expands its value directly into the DOM.
Interpolations like %...% work in both printed text and attribute values.
Note: an undefined name expands to an empty string.

---

## 2) Core Directives  

### **`*print` - Output text content**
Replaces the element’s text with the variable value.  

```html
<h1 *print="'Hello world!'"></h1>

<h1>Hello world!</h1>
^ rendered output
> *print - Output text content.
```

Equivalent to writing `%var%` inside, but easier to read when nesting.
```html
<h1>%message%</h1>

<h1></h1>
^ rendered output
> %message% - Output text content.
It vanished - the variable isn’t defined yet. That’s expected. Let’s go to the next step.
```

---

### **`*let` - Define a local variable**  
Defines a scoped variable for the current element and its children.  
Works in text nodes **and** attributes.  

```html
<h1 *let="message='Hello Sercrod!'">%message%</h1>

<h1>Hello Sercrod!</h1>
^ rendered output
> %message% - Output text content.
```

The variable defined by *let can be used in any child element.

You can also...:
```html
<h1 *let="message='Hello Sercrod!'" *print="message"></h1>

<h1>Hello Sercrod!</h1>
^ rendered output
> *print - Output text content.
```

You can also use multiple variables:
```html
<p *let="x=2, y=3">Sum: %x + y%</p>

<p>Sum: 5</p>
^ rendered output
> %...% evaluates JavaScript expressions. "x + y" is calculated, not concatenated.

<p *let="x=2, y=3">Sum: %{x + y}%</p>

<p>Sum: 5</p>
^ rendered output
Same result - braces make it explicit that this is an expression.
```

`*let` can also hold arrays or objects.

```html
<ul *let="fruits=['Apple','Banana','Cherry']">
  <li>%fruits[0]%</li>
  <li>%fruits[1]%</li>
  <li>%fruits[2]%</li>
</ul>

<ul>
  <li>Apple</li>
  <li>Banana</li>
  <li>Cherry</li>
</ul>
^ rendered output
> `*let` - Supports arrays and objects.
```


---

### `*if` - Conditional display
Shows the element only when the given expression is true.

```html
<p *let="x = 2" *if="x == 2">Collect.</p>

<p>Collect.</p>
^ rendered output
> `*if` - Evaluates the expression. The element appears only if the condition is true.

<p *let="x = 2" *if="x == 1">Not collect.</p>


^ rendered output
It vanished whole P tag. - That’s expected.
The element is completely removed from the DOM.
```

You can combine `*if` with another one as an else case.

---

### `*for` - Loop through an array
Repeats the element for each item in an array.

```html
<ul *let="fruits=['Apple','Banana','Cherry']">
  <li *for="fruit of fruits">%fruit%</li>
</ul>

<ul>
  <li>Apple</li>
  <li>Banana</li>
  <li>Cherry</li>
</ul>
^ rendered output
> `*for` - Iterates over arrays using the `of` keyword.
```

You can also access the index if needed.

```html
<ul *let="fruits=['Apple','Banana','Cherry']">
  <li *for="(i,fruit) of fruits">%i% - %fruit%</li>
</ul>

<ul>
  <li>0 - Apple</li>
  <li>1 - Banana</li>
  <li>2 - Cherry</li>
</ul>
^ rendered output
> `*for` - Supports both index and value.  
> `in` can also be used, but Sercrod will warn about deprecated syntax.
```

Legacy style (keys via `in`):

```html
<ul *let="fruits=['Apple','Banana','Cherry']">
  <li *for="i in fruits">%fruits[i]%</li>
</ul>

<ul>
  <li>Apple</li>
  <li>Banana</li>
  <li>Cherry</li>
</ul>
^ rendered output
> Legacy: `in` enumerates keys (indices). Prefer `of` for values; use `(i,fruit) of fruits` when you need both.
```

---

### `*input` - Read and write form values
Connects a form control’s value with a variable. Works with `<input>`, `<textarea>`, and `<select>`.
When the user edits the field, the variable updates.
When the variable changes, the field reflects it.  
_Note: This example uses a Sercrod host (`data={}`) to store the value.
You’ll learn more about `data` in the next section._

```html
<input *input="name">
<p>Hello, %name%!</p>

<input *input="name">
<p>Hello, !</p>
^ rendered output (before typing)

<input *input="name">
<p>Hello, Sercrod!</p>
^ rendered output (after confirming input "Sercrod")

```

Textareas behave the same:

```html
<serc-rod>
  <textarea *input="note"></textarea>
  <p>%note%</p>
<serc-rod>

<serc-rod>
  <textarea *input="note"></textarea>
  <p>%note%</p>
<serc-rod>
^ rendered output (before typing)

<serc-rod>
  <textarea *input="note">Short message.</textarea>
  <p>Short message.</p>
<serc-rod>
^ rendered output (after confirming input "Short message.")
```

Textareas can also display values directly:

```html
<textarea *let="note='Short message.'">%note%</textarea>

<textarea>Short message.</textarea>
^ rendered output
```

> `*let` defines a variable, and `%note%` writes it inside the textarea content.
---

## 5) Summary  

| Directive | Purpose | Example | Result |
|------------|----------|----------|--------|
| `*let` | Define local variable | `<p *let="x=5">%x%</p>` | `5` |
| `*print` | Output text content | `<h1 *print="title"></h1>` | Displays title |
| `*if` | Conditional display | `<p *if="flag">Yes</p>` | Only if `flag` |
| `*for` | Loop with index | `<li *for="(i,x) in list">%i%: %x%</li>` | Indexed list |
| `*input` | User input (reflects variable) | `<input *input="name" />` | Updates `name` live |

---

Next chapter: **Reactivity and Observe Modes** (`docs/3-reactivity.en.md`)  
