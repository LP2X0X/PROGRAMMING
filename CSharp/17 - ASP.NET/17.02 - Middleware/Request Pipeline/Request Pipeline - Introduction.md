---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---


## Overview

The **request pipeline** in ASP.NET Core is the sequence of middleware components that process every HTTP request and response. Each middleware component can inspect, modify, short-circuit, or pass the request to the next component. The order in which middleware is registered in `Program.cs` directly determines how requests flow through the application -- and getting the order wrong is one of the most common sources of subtle bugs in ASP.NET Core applications.

Understanding the pipeline is not optional. It is the backbone of how ASP.NET Core handles cross-cutting concerns like authentication, error handling, logging, and routing.
