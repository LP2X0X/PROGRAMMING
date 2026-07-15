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

### MVC vs MVVM

These rendering strategies align with two different **architectural patterns** for separating concerns:

| | MVC (Model-View-Controller) | MVVM (Model-View-ViewModel) |
|---|---|---|
| **Middleman** | Controller | View Model |
| **Data flow** | One-way — controller pushes data to view, view renders once | Two-way — view and view model update each other through bindings |
| **View interaction** | Controller receives requests, picks a view | View binds to view model properties and commands |
| **Middleman knows about View?** | Yes — controller selects which view to render | No — view model has no reference to the view |
| **Lifecycle** | Stateless — created per request, then dies | Stateful — lives as long as the view is open |
| **Used in** | ASP.NET MVC, Razor Pages, Rails, Django | WPF, MAUI, Blazor, Xamarin, SwiftUI |

MVC fits **server-side rendering** naturally — each HTTP request creates a controller, the controller processes it and returns a view, then the controller dies. MVVM fits **client-side and desktop apps** where the UI stays alive and reacts to changes through data binding.

### The View Model Difference: MVC vs MVVM

In MVC, a view model is a **data bag** — a projection of the domain model shaped for a specific screen. Like a printed report: generated once, read, done.

In MVVM, the view model is the **view's brain** — it holds data, state, behavior, and commands. It *is* the view, minus the visuals. You could describe the entire screen from the view model alone without ever seeing the XAML.

```csharp
// MVVM View Model — IS the login screen without pixels
public class LoginViewModel : INotifyPropertyChanged
{
    // Data
    public string Email { get; set; }
    public string Password { get; set; }

    // State
    public bool IsLoading { get; set; }
    public bool IsLoginEnabled => !string.IsNullOrEmpty(Email) 
                                && !string.IsNullOrEmpty(Password) 
                                && !IsLoading;
    public string ErrorMessage { get; set; }

    // Behavior
    public ICommand LoginCommand { get; }
    public ICommand ForgotPasswordCommand { get; }
}
```

The XAML view is just the **skin** — it decides where and how to render what the view model already defines through data binding.

| | MVC View Model | MVVM View Model |
|---|---|---|
| **What it is** | Data bag — a projection of the model | The view's brain — data + state + behavior |
| **Analogy** | A printed report | A live dashboard |
| **Behavior?** | No — just properties | Yes — commands, validation, state logic |
| **Reacts to changes?** | No — rendered once, thrown away | Yes — `INotifyPropertyChanged`, two-way binding |
| **Testable as?** | Just check property values | Can simulate the entire UI flow without a view |

> [!ad-note] Desktop Developers -- Mental Model
> If you come from WinForms or WPF, CSR will feel more familiar -- the JavaScript framework works like a UI toolkit (components, state, event handlers). SSR is more like generating a printable document on every user action. In neither case does your server-side C# directly render pixels -- the **browser** is always the GUI runtime.

> [!summary] Section Summary
> - **SSR** (Razor Pages, MVC): server builds complete HTML, browser just displays it. Simpler, one C# codebase, great SEO.
> - **CSR** (Web APIs + React/Vue/Angular): server sends JSON, browser builds the UI. More interactive, but two codebases.
> - **Blazor** bridges both -- C# in the browser (WebAssembly) or server-pushed updates (Server/Auto).
