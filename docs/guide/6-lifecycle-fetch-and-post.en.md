# Lifecycle (advanced)
_Last updated: 2025-10-22_

>This chapter introduces Sercrod’s local lifecycle features that let you save and restore data as JSON files directly in the browser.  

In the previous chapter, you learned how *save and *load handled data persistence inside the browser.  
This section extends that same lifecycle outward ? connecting Sercrod to remote data sources.

*fetch retrieves JSON data from a server,  
while *post sends the current Sercrod data back to it.

First, we will send the current Sercrod data to the server with *post and persist it as a JSON file.  
Next, we will fetch that same file with *fetch to restore the component state.

## *post: Save on the server

Use *post to transmit the component’s current data to an endpoint that writes it to storage.  
When triggered, Sercrod serializes the data as JSON and sends it over HTTP.  
Once the server saves it, the resulting JSON file will be used by *fetch in the next step.

# Lifecycle - *post (server upload)

> Lead line  
> *post is the directive that sends Sercrod’s current data to a server and saves it as a JSON file.

## 30-second example
```html
<serc-rod data='{"question":"What is your name?","answer":"Sercrod."}'>
  <p>Q: %question%</p>
  <p>A: %answer%</p>

  <!-- Send current data to the server -->
  <button *post="/api/save">Send</button>
</serc-rod>
```
^ Rendered output (omit <serc-rod>)
```html
<p>Q: What is your name?</p>
<p>A: Sercrod.</p>
<button>Send</button>
```

What happens  
1. You click the **Send** button.  
2. Sercrod converts the current data into JSON format.  
3. The JSON is sent to `/api/save` via HTTP POST.  
4. The server receives it and writes it as a file (for example, `data.json`).  
5. After completion, Sercrod fires a `sercrod-posted` event.

---

## Server-side examples

If you are familiar with backend development,  
you can handle Sercrod’s *post request as a normal JSON upload.  
Others may skip this section.

---

The Node.js script below should be saved as the file that runs at **"/api/"**.

### Node.js (Express)
```js
import express from "express";
import fs from "fs";
import path from "path";

const app = express();
app.use(express.json());

app.post("/api/", (req, res)=>{
  const file = path.join(path.dirname(new URL(import.meta.url).pathname), "data.json");
  fs.writeFileSync(file, JSON.stringify(req.body, null, 2));
  res.json({ status: "ok", saved: "data.json" });
});

app.listen(3000, ()=>console.log("Server running on http://localhost:3000"));
```

---

The PHP script below should be saved as the file that runs at **"/api/"**.

### PHP
```php
<?php
// Runs at /api/
$json = file_get_contents("php://input");
$data = json_decode($json, true);
$file = __DIR__ . "/data.json";
file_put_contents($file, json_encode($data, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));
echo json_encode(["status"=>"ok","saved"=>"data.json"]);
```
---

The Python script below should be saved as the file that runs at **"/api/"**.

### Python (Flask)
```python
from flask import Flask, request, jsonify
import json, os

app = Flask(__name__)

@app.route("/api/", methods=["POST"])
def post_data():
    data = request.get_json()
    if not data:
        return jsonify(status="error", message="invalid JSON"), 400
    with open(os.path.join(os.path.dirname(__file__), "data.json"), "w", encoding="utf-8") as f:
        json.dump(data, f, indent=2, ensure_ascii=False)
    return jsonify(status="ok", saved="data.json")

if __name__ == "__main__":
    app.run(port=3000)
```

---

Each example writes the received JSON into a file named `data.json`.  
That same file will later be fetched by *fetch to restore the Sercrod element.

---

## Resulting file
When the server receives the data, it saves it as a JSON file.

```json
{ "question": "What is your name?", "answer": "Sercrod." }
```

This file will later be fetched by the *fetch directive to restore the same state.

Before using *fetch, make sure that a data.json file exists
- either one created by *post on the server, or one you prepared manually.
For local testing, create an api folder next to your HTML file and put data.json inside it.

# Lifecycle - *fetch (server import)

> Lead line  
> *fetch loads JSON data from a server and applies it to the Sercrod element, completing the round trip that began with *post.

Before using *fetch, make sure that a `data.json` file exists  
- either one created by *post on the server, or one you prepared manually.  
For local testing, create an `api` folder next to your HTML file and put `data.json` inside it.

## 30-second example
```html
<serc-rod *fetch="/api/data.json">
  <p *print="`Q: ${question}`">Q: What is the present?</p>
  <p *print="`A: ${answer}`">A: Instant.</p>
</serc-rod>
```

## Result before fetching
When the page first loads, the Sercrod element is empty.

```html
<p>Q: What is the present?</p>
<p>A: Instant.</p>
```
^ Rendered output before fetching (omit <serc-rod>)
This placeholder appears only for a moment and usually cannot be seen.

---

## Result after fetching
After Sercrod fetches `/api/data.json`, the values from that file appear automatically.

## Result after fetching
After Sercrod fetches `/api/data.json`, values from that file replace the placeholders.

```html
<p>Q: What is your name?</p>
<p>A: Sercrod.</p>
```
^ Rendered output after fetching (omit <serc-rod>)

## What happens
1. When the Sercrod element is created, it sends an HTTP GET request to `/api/data.json`.  
2. If the file exists, the JSON data is fetched and merged into the element’s data context.  
3. The UI automatically re-renders to reflect the new values.  
4. After completion, Sercrod fires a `sercrod-fetched` event.

---

## Summary
The *post and *fetch directives extend Sercrod’s local lifecycle to the server.

| Action | Direction | Typical file | Event |
|---------|------------|---------------|--------|
| *post | Send data to the server | `/api/data.json` | `sercrod-posted` |
| *fetch | Load data from the server | `/api/data.json` | `sercrod-fetched` |

You can save data on the server with *post and bring it back later with *fetch.  
The format is JSON. Both work with the same structure as *save and *load,  
so data can move freely between local files and remote servers.

In real projects, you will often start by preparing a JSON file first and loading it with *fetch.  
This chapter used the *post -> *fetch order for clarity, showing how the same data can travel both ways.

This completes the full round trip of Sercrod’s lifecycle.
