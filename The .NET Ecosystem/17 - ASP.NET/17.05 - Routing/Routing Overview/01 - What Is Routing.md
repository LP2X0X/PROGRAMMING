---
tags:
  - csharp
  - asp-net-core
  - routing
  - fundamentals
---


**Routing** is the process of matching an incoming HTTP request (its URL path, HTTP method, and sometimes other data) to an **endpoint handler** -- the code that will process the request and produce a response.

In ASP.NET Core, endpoint handlers can be:

- **Controller actions** (MVC pattern)
- **Razor Page handlers** (page-based model)
- **Minimal API delegates** (lambda-based endpoints introduced in .NET 6)
- **gRPC services**, **SignalR hubs**, **health check endpoints**, and more

Routing answers the question: *"Given this incoming URL and HTTP method, which piece of code should handle it?"*

> [!ad-note] Definition
> **Endpoint**: A unit of request-handling logic in ASP.NET Core. Each endpoint has a **route pattern** (the URL template), **metadata** (authorization policies, CORS, etc.), and a **request delegate** (the code that runs).

### Why Routing Matters

Without routing, you would have to write manual URL parsing logic in every application. Routing provides:

- **Declarative URL patterns** -- define what URLs look like, not how to parse them
- **Parameter extraction** -- automatically pull values from URL segments into method parameters
- **URL generation** -- create URLs from route data (the reverse of matching)
- **Centralized configuration** -- all URL-to-handler mappings are visible in one place
- **Middleware integration** -- authorization, CORS, and rate limiting can inspect the selected endpoint before it executes

### Routing Styles Comparison

| Routing Style | Typical Use | Advantages | Disadvantages |
|---|---|---|---|
| **Convention-based routes** | HTML-generating MVC controllers | Terse definition in one location. Forces a consistent layout of MVC controllers. | Routes are defined in a different place from your controllers. Overriding conventions can be tricky and error-prone. Adds an extra layer of indirection when routing a request. |
| **Explicit routes** (attribute/minimal API) | Minimal API endpoints, Web API MVC controllers | Gives complete control of route templates for every endpoint. Routes are defined next to the endpoint they execute. | Verbose compared with convention-based routing. Can be easy to overcustomize route templates. Route templates may be scattered throughout your application. |
| **Convention-based generation of explicit routes** | Razor Pages | Encourages consistent set of exposed URLs. Terse when you stick to conventions. Easily override the route template for a single page. Customize conventions globally to change exposed URLs. | Possible to overcustomize route templates. You must calculate what the route template for a page is, rather than its being explicitly defined in your app. |

> [!summary] Section Summary
> - Routing matches HTTP requests to endpoint handlers based on URL patterns.
> - Endpoints include controller actions, Razor Pages, minimal APIs, gRPC, SignalR, and more.
> - Routing provides parameter extraction, URL generation, and middleware integration.
> - Each endpoint has a route pattern, metadata, and a request delegate.
