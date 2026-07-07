---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


Understanding the pipeline a request travels through is essential. Here is the order of operations when a request hits a controller action:

```
Client sends HTTP request
       |
       v
1. Routing selects controller + action         --> see [[17.05 - Routing]]
       |
       v
2. Controller is instantiated (DI resolves
   constructor parameters)                     --> see [[17.03 - Dependency Injection]]
       |
       v
3. Model binding maps request data
   (route, query, body, headers, form)
   to action parameters                        --> see [[Model Binding]]
       |
       v
4. Filters execute (authorization, resource,
   action, exception filters)                  --> see [[Filters]]
       |
       v
5. Action method executes
       |
       v
6. Action result executes
   (serializes response body, sets status)     --> see [[Action Results]]
       |
       v
7. Response sent to client
```

```ad-note
If model validation fails (with `[ApiController]`), the pipeline short-circuits at step 3 and returns a 400 response. The action method never executes.
```
