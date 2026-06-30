---
tags: [csharp, asp-net-core, fundamentals, web]
---


The application models above fall into two rendering strategies. Understanding this distinction is fundamental to choosing the right approach.

### Server-Side Rendering (SSR)

With **Razor Pages** and **MVC Controllers**, your C# code on the server builds the complete HTML before sending it to the browser. The browser receives a finished document and simply displays it.

```
Browser: "GET /products/5"
    │
    ▼
Server:  1. Fetches product from database
         2. Runs Razor template with the data
         3. Produces complete HTML string
         4. Sends finished HTML back
    │
    ▼
Browser: Receives HTML, displays it immediately
```

### Client-Side Rendering (CSR)

With **Web API Controllers** and **Minimal APIs**, the server only returns JSON data. A JavaScript framework (React, Vue, Angular) running in the browser receives that data and builds the HTML itself.

```
Browser: Loads a JavaScript app (React, Vue, etc.)
React:   "GET /api/products/5"
    │
    ▼
Server:  1. Fetches product from database
         2. Returns JSON: {"name":"Mouse","price":29.99}
    │
    ▼
React:   Receives JSON, builds the HTML in the browser
```

### Comparison

| | SSR (Razor Pages / MVC) | CSR (API + JS Framework) |
|---|---|---|
| **Who builds HTML?** | Server (C#) | Browser (JavaScript) |
| **First page load** | Fast -- HTML arrives ready to display | Slower -- must download the JS app first |
| **Interactivity** | Each action = full page reload | Smooth, app-like experience |
| **SEO** | Great -- search engines see complete HTML | Harder -- requires extra setup |
| **Complexity** | One codebase (C#) | Two codebases (C# API + JS frontend) |
| **Feels like** | Traditional website | Desktop/mobile app in the browser |
| **Best for** | Content sites, forms, admin panels | Dashboards, highly interactive UIs |

> [!ad-note] Blazor Blurs the Line
> Blazor does not fit neatly into either category. **Blazor Server** renders on the server but pushes incremental UI updates over a WebSocket (SignalR) -- no full page reloads. **Blazor WebAssembly** runs C# directly in the browser, making it CSR but without JavaScript. **Blazor Auto** (.NET 8+) starts with server rendering for fast first load, then switches to WebAssembly for interactivity.

> [!ad-note] Desktop Developers -- Mental Model
> If you come from WinForms or WPF, CSR will feel more familiar -- the JavaScript framework works like a UI toolkit (components, state, event handlers). SSR is more like generating a printable document on every user action. In neither case does your server-side C# directly render pixels -- the **browser** is always the GUI runtime.

> [!summary] Section Summary
> - **SSR** (Razor Pages, MVC): server builds complete HTML, browser just displays it. Simpler, one C# codebase, great SEO.
> - **CSR** (Web APIs + React/Vue/Angular): server sends JSON, browser builds the UI. More interactive, but two codebases.
> - **Blazor** bridges both -- C# in the browser (WebAssembly) or server-pushed updates (Server/Auto).
