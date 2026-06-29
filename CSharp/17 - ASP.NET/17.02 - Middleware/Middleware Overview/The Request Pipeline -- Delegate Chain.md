---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## The Request Pipeline -- Delegate Chain

The ASP.NET Core request pipeline is built as a **chain of request delegates**. Each delegate represents one middleware component. When a request arrives, the framework invokes the first delegate, which optionally invokes the next, and so on -- forming a chain.

Here is an ASCII diagram showing how a request flows through three middleware components and back:

```
                        ASP.NET Core Request Pipeline

  HTTP Request
       |
       v
  +--------------------+
  | Middleware 1        |
  |  (Logging)          |
  |                     |
  |  Before next() -----+--->  +--------------------+
  |                     |      | Middleware 2        |
  |                     |      |  (Authentication)   |
  |                     |      |                     |
  |                     |      |  Before next() -----+--->  +--------------------+
  |                     |      |                     |      | Middleware 3        |
  |                     |      |                     |      |  (Routing/Endpoint) |
  |                     |      |                     |      |                     |
  |                     |      |                     |      |  Generates Response |
  |                     |      |                     |      |                     |
  |                     |      |  After next()  <----+------|  (terminal)         |
  |                     |      |                     |      +--------------------+
  |  After next()  <----+------+                     |
  |                     |      +--------------------+
  +--------------------+
       |
       v
  HTTP Response
```

And here is a simplified linear view:

```
  Request  --->  [Logging] ---> [Auth] ---> [CORS] ---> [Routing] ---> [Endpoint]
  Response <---  [Logging] <--- [Auth] <--- [CORS] <--- [Routing] <--- [Endpoint]
```

Each arrow marked `next()` is a call to the **next delegate** in the chain. The last middleware (often the endpoint/routing middleware) generates the actual response, and then control flows back through each middleware in reverse order.

> [!ad-note]
> The pipeline is constructed at application startup in `Program.cs` (or `Startup.cs` in older projects). Once built, the delegate chain is immutable -- it does not change per request.

> [!summary] Section Summary
> The request pipeline is a chain of delegates where each middleware calls `next()` to invoke the next one. The request flows forward through the chain, and the response flows backward. The chain is built at startup and remains fixed for the lifetime of the application.
