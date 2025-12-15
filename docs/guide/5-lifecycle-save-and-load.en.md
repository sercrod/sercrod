# Lifecycle (basic)
_Last updated: 2025-10-22_

> This chapter introduces Sercrod’s local lifecycle features that let you save and restore data as JSON files directly in the browser.  

## 30-second example
```html
<serc-rod data='{"question":"What is the present?","answer":"Instant."}'>
  <p>Q: %question%</p>
  <p>A: %answer%</p>

  <!-- Save current data as a JSON file -->
  <button *save>Save</button>
</serc-rod>
```
^ Rendered output (omit <serc-rod>)
```html
<p>Q: What is the present?</p>
<p>A: Instant.</p>
<button>Save</button>
```

What happens  
1. You click the **Save** button.  
2. Sercrod converts the current data to JSON format.  
3. The browser automatically downloads it as a `.json` file.

---

## Resulting JSON file
Create this JSON file and download it through the *save button.
```json
{ "question": "What is the present?", "answer": "Instant." }
```

This file can later be reloaded with *load.

---

## Beginner notes
- The file format is JSON.  
- The saved data reflects the current state of the Sercrod element.  
- The browser will usually name it something like `Sercrod-20251021-151515.json`.  
- Saving happens entirely in the browser; no data is sent anywhere.  

---

## How it fits your page
- Use `*save` on a button or clickable element.  
- When clicked, Sercrod serializes the element’s data and downloads it.  
- Pair it with `*load` to restore the same state later.  

---

## Event
- Fires a `sercrod-saved` event after completion.  
- The event’s `detail` object includes a `json` field containing the serialized data.
- For complete event details, see the Event Reference.  

---

Once you can save, you’ll soon wonder how to bring it back.
That’s where *load begins.

---

# Lifecycle - *load (local file import)

> Lead line  
> *load is the directive that applies a user-chosen local file to the current data.

## 30-second example
```html
<serc-rod data='{"question":"What is the past?","answer":"Memory."}'>
  <p>Q: %question%</p>
  <p>A: %answer%</p>

  <!-- Pick a JSON file and apply it to this page's data -->
  <input type="file" accept="application/json" *load>
</serc-rod>

<p>Q: What is the past?</p>
<p>A: Memory.</p>
<input type="file">
^ Rendered output before loading (omit <serc-rod>)
```

What happens  
1. You choose a file such as **Sercrod-20251021-151515.json**.  
2. Sercrod reads it as JSON and updates the data inside this Sercrod element.  
3. The UI automatically re-renders with the new values.

---

## Try loading the file you saved earlier
Use the JSON file you created with *save in the previous section.  
Upload that file through the *load input ? it restores the same “present” state you saved.

```html
<p>Q: What is the present?</p>
<p>A: Instant.</p>
^ Rendered output after loading (omit <serc-rod>)
```

## Try a new JSON file
Create this JSON file and upload it through the *load input.  
```json
{ "question": "What is the future?", "answer": 42 }
```

### Result after loading
```html
<p>Q: What is the future?</p>
<p>A: 42</p>
```
^ Rendered output after loading (omit <serc-rod>)

---

## Beginner notes
- Supported format: JSON is assumed.  
- Only keys that match your bindings are applied.  
- If the JSON is invalid or unreadable, a warning appears and the UI stays unchanged.  
- All processing occurs locally in the browser. No data is uploaded.

---

## How it fits your page
- Use `*load` on an `<input type="file">` or a button that opens a file picker.  
- After loading, bound values (`%key%`, `*print`, etc.) update automatically.  
- You can pair it with `*save` to round-trip data between “save to file” and “load from file.”

---

## Spec details ? confirmed facts only
### Completion event
- Event name: `sercrod-loaded`  
- Target: the Sercrod element that performed the load  
- Bubbling: `bubbles: true`  
- Cross shadow boundary: `composed: true`  
- `detail` payload:
  - `host`: the originating Sercrod element  
  - `fileName`: selected file name, or `null`  
  - `props`: additional info from implementation, or `null`  
  - `json`: the parsed JSON object

### Errors and warnings
- When JSON is invalid or cannot be applied, show a warning and do not change the UI.  
- Follow the project convention using the `[Sercrod warn]` prefix.

---

## Pending items
- Exact apply scope and overwrite vs deep-merge policy  
- Options for partial application  
- Support for additional file types and mapping rules


Once you can load, the flow between data and page becomes complete.  
From here, Sercrod begins to move with your data.

## Summary
The *save and *load directives form the local cycle of Sercrod’s lifecycle.  
You save a moment, then bring it back ? data and view stay in harmony.  
Next, *fetch and *post will extend this flow beyond your browser.
