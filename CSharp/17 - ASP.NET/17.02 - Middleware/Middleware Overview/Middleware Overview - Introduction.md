---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---


**Middleware** is a fundamental building block in ASP.NET Core that forms the **request processing pipeline**. Every HTTP request that enters your application flows through a series of middleware components, each of which can inspect, modify, or short-circuit the request before it reaches your application logic -- and do the same on the way back out with the response.

Understanding middleware is essential because it controls everything that happens to a request: logging, authentication, authorization, error handling, routing, CORS, compression, and more. If something goes wrong in your pipeline, it is almost always a middleware ordering issue.
