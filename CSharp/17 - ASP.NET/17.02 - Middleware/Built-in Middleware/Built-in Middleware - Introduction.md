---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---


ASP.NET Core ships with a rich set of **built-in middleware** components that handle cross-cutting concerns such as error handling, security, routing, caching, and compression. Each middleware is a discrete component in the [[Request Pipeline]] that processes HTTP requests and responses in a specific order. Understanding what each built-in middleware does, where it belongs in the pipeline, and how to configure it is essential for building robust web applications.

This note provides a complete reference for the most important built-in middleware components. For foundational concepts, see [[Middleware Overview]]. For writing your own components, see [[Custom Middleware]].
