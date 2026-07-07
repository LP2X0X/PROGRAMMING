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

> [!summary] Section Summary
> - Routing matches HTTP requests to endpoint handlers based on URL patterns.
> - Endpoints include controller actions, Razor Pages, minimal APIs, gRPC, SignalR, and more.
> - Routing provides parameter extraction, URL generation, and middleware integration.
> - Each endpoint has a route pattern, metadata, and a request delegate.
