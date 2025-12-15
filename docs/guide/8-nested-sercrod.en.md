# Nested Sercrod  
_Last updated: 2025-10-22_

> This chapter explains how Sercrod behaves when multiple `<serc-rod>` elements are nested inside one another.  
> You will learn how each Sercrod keeps its own data scope and how they interact.

---

## 1. Overview

Sometimes you may want to use more than one `<serc-rod>` on the same page.  
For example, an outer Sercrod might hold page-wide data, while an inner Sercrod handles a smaller section such as a comment box or a sidebar.  

Sercrod supports this naturally ? every `<serc-rod>` works as an independent world with its own data and lifecycle.  
This means that updates in one Sercrod do not interfere with others, even when they are nested.

## 2. Different update timing

Nested Sercrod behaves differently from a single Sercrod when you use input bindings.

Compare the following two cases.

---

### Case 1: Single Sercrod

```html
<serc-rod data='{"text":"update when confirmed"}'>
  <input *input="text">
  <p>%text%</p>
</serc-rod>

<p>update when confirmed</p>
^ Rendered output (omit `<serc-rod>`)
```

Here, the `<p>` element updates **only after the input is confirmed**.  
While typing, the value inside `<p>` does not change.

---

### Case 2: Nested Sercrod

```html
<serc-rod data='{"text":"update while typing"}'>
  <input *input="text">
  <serc-rod>
    <p>%text%</p>
  </serc-rod>
</serc-rod>

<p>update while typing</p>
^ Rendered output (omit `<serc-rod>`)
```

When a second `<serc-rod>` is placed inside, the behavior changes:  
the inner Sercrod shows the same value as you type, updating simultaneously with the input field.

---

The two cases look similar, but their timing differs:
- **Single Sercrod:** waits until the input event completes  
- **Nested Sercrod:** updates instantly with every keystroke

## 3. Independent scopes

Each `<serc-rod>` works with its own data area.  
Even when nested, inner and outer Sercrod do not automatically share values ?  
each one manages its own copy of data.

```html
<serc-rod data='{"text":"We are the world."}'>
  <h2 *print="text"></h2>

  <serc-rod data='{"text":"We are the children."}'>
    <p *print="text"></p>
  </serc-rod>
</serc-rod>

<h2>We are the world.</h2>
<p>We are the children.</p>
^ Rendered output (omit `<serc-rod>`)
```

Here both Sercrod use the same property name (`text`),  
but they belong to separate data areas.  
Changing the outer value will not affect the inner one, and vice versa.

This separation lets you build multiple self-contained parts on the same page  
without worrying about variable conflicts.

---

## 4. Data inheritance and separation

When Sercrod elements are nested,  
the child Sercrod can use the parent's data as long as it does not define its own.  
Once the child declares its own `data`,  
it becomes completely independent ? the parent's values are no longer visible.

---

### Case 1: Shared scope 

```html
<serc-rod data='{"word":"floccinaucinihilipilification"}'>
  <p>The word meaning 'to regard something as worthless' is %word%. Repeat.</p>
  <serc-rod>
    <p>The word is %word%.</p>
  </serc-rod>
</serc-rod>

<p>The word meaning 'to regard something as worthless' is floccinaucinihilipilification. Repeat.</p>
<p>The word is floccinaucinihilipilification.</p>
^ Rendered output (omit `<serc-rod>`)
```

Here the inner Sercrod defines no `data`,  
so it repeats the same word,  
echoing perfectly inside the same world.

---

### Case 2: Isolated scope

```html
<serc-rod data='{"word":"floccinaucinihilipilification"}'>
  <p>The word meaning 'to regard something as worthless' is %word%. Repeat.</p>
  <serc-rod data='{"food":"focaccia"}'>
    <p>The word is %word%... I want to eat a %food%.</p>
  </serc-rod>
</serc-rod>

<p>The word meaning 'to regard something as worthless' is floccinaucinihilipilification. Repeat.</p>
<p>The word is ... I want to eat a focaccia.</p>
^ Rendered output (omit `<serc-rod>`)
```

When the inner Sercrod defines its own `data`,  
it no longer refers to the outer `word`.  
Only the variables inside its own `data` remain visible,  
so `%word%` shows nothing while `%food%` appears normally.

---

Sercrod's nested design keeps each element independent by default.  
Only when the child omits its own `data` does it rely on the parent's values.  
This separation makes nested components easy to reason about and prevents hidden dependencies.

---

## 5. Common mistakes

When working with nested Sercrod, beginners sometimes assume that data automatically flows between them.  
In practice, each Sercrod controls its own scope.

| Mistake | What happens | Correct understanding |
|----------|---------------|------------------------|
| Expecting automatic sharing | The inner Sercrod can access parent data only when it has no `data` of its own. Beginners sometimes expect both to stay linked even after defining child `data`. | Data is inherited automatically only when the child defines no `data`. Once defined, it becomes a separate scope. |
| Expecting parent values after defining child `data` | After the child defines its own `data`, variables from the parent become unavailable. | A new `data` creates a new scope. Only the child's variables remain visible. |
| Trying to update parent data | Changes made inside the inner Sercrod do not affect the outer one. | Every Sercrod manages its own data independently. |

These behaviors are intentional.  
By keeping data local to each Sercrod,  
the framework avoids accidental side effects and makes nested structures predictable.

---

## 6. let inside nested scopes

When `*let` is used inside a nested Sercrod,  
it can **read** values created by the parent's `*let`,  
but it **cannot modify** them.  
Each Sercrod still maintains its own local context.

```html
<serc-rod *let="count=1">
  <p>Outer: %count%</p>
  <serc-rod *let="count=count+1">
    <p>Inner: %count%</p>
  </serc-rod>
</serc-rod>

<p>Outer: 1</p>
<p>Inner: 2</p>
^ Rendered output (omit `<serc-rod>`)
```

Here the inner Sercrod reads the parent's `count`  
and creates a new one based on it.  
The parent's value remains unchanged.

In Sercrod, `*let` works like a bridge that copies values at creation time,  
not a live link between scopes.  
The next chapter explains how to connect them more directly.

---

## 7. Summary

Nested Sercrod shows how each **Sercrod context** (whether `<serc-rod>` or another host) forms its own scope.  
Key points:

- Each context manages its own `data` and lifecycle.  
- A child context inherits the parent's values **only when it defines no `data` of its own**.  
- Once the child defines `data`, the two contexts are **isolated**.  
- Updates inside the child do **not** affect the parent.  
- `*let` inside a child can **read** values introduced by the parent's `*let`, but **cannot modify** them.
