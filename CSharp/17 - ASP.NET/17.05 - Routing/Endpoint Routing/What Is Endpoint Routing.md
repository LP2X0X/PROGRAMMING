---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


**Endpoint routing** is the routing architecture introduced in .NET Core 3.0 that replaced the older, MVC-internal routing system. It provides a **unified routing model** that works the same way for controllers, Razor Pages, minimal APIs, gRPC services, SignalR hubs, health checks, and any custom endpoints.

### The Problem It Solved

Before endpoint routing (ASP.NET Core 1.x and 2.x):
- Routing lived *inside* MVC. The middleware pipeline had no visibility into which action would handle a request.
- Authorization middleware could not check endpoint-specific policies because the endpoint had not been selected yet.
- CORS decisions were made inside MVC filters, not at the middleware level.
- Different endpoint types (controllers vs. SignalR) had separate routing implementations.

### The Solution

Endpoint routing extracts route matching out of MVC and into the **middleware pipeline** itself. This means:
- Any middleware can know which endpoint was selected
- Authorization, CORS, rate limiting, and other cross-cutting concerns work uniformly
- One routing system serves all endpoint types
- The routing decision is visible to the entire pipeline, not hidden inside a single framework

> [!ad-note] Definition
> **Endpoint**: An object with three key properties:
> - **RequestDelegate**: The code that handles the request
> - **RoutePattern**: The URL template that matches incoming requests
> - **Metadata**: A collection of objects (authorization policies, CORS policies, etc.) that middleware can inspect

> [!summary] Section Summary
> - Endpoint routing is the unified routing system since .NET Core 3.0.
> - It decouples route matching from endpoint execution, enabling middleware inspection.
> - Before endpoint routing, route matching was hidden inside MVC with no pipeline visibility.
> - All endpoint types (controllers, Razor Pages, minimal APIs, gRPC, SignalR) share one routing system.
