---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
---


**Custom middleware** is user-defined code that plugs into the ASP.NET Core [[Request Pipeline]] to inspect, modify, or short-circuit HTTP requests and responses. While ASP.NET Core ships with many [[Built-in Middleware]] components (authentication, static files, routing, etc.), real-world applications almost always need custom middleware for cross-cutting concerns like request logging, correlation IDs, global error handling, or API key validation.

This note covers the three ways to create custom middleware, how dependency injection interacts with middleware lifetime, real-world implementations, testing strategies, and when to choose middleware over action filters.

> [!ad-note]
> This note assumes familiarity with the [[Middleware Overview]] and [[Request Pipeline]] concepts. Custom middleware sits at the same level as built-in middleware -- the pipeline does not distinguish between them.
