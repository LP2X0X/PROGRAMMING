---
tags:
  - csharp
  - asp-net-core
  - http
  - status-codes
---

## Comprehensive Summary

> [!ad-tip] Complete Summary
> **HTTP status codes** are three-digit numbers in every HTTP response that tell the client what happened. They are the primary mechanism for communicating outcomes in web APIs -- replacing the return values and exceptions you are used to from desktop development.
>
> **The five classes** are determined by the first digit:
> - **1xx (Informational)**: Server acknowledgments, handled automatically by the HTTP infrastructure. You rarely interact with these.
> - **2xx (Success)**: The request succeeded. Use **200 OK** for standard responses, **201 Created** when a resource is created (with a `Location` header), **204 No Content** for successful operations with no body (DELETE, PUT), and **202 Accepted** for queued/async operations.
> - **3xx (Redirection)**: The client should follow a different URL. **301** for permanent moves, **302** for temporary redirects, **304** for cached responses. The Post-Redirect-Get pattern (POST then redirect to GET) prevents duplicate form submissions.
> - **4xx (Client Error)**: The client sent a bad request. **400** for invalid data, **401** for "not authenticated" (despite its misleading name), **403** for "authenticated but not authorized," **404** for resources that do not exist, **409** for state conflicts, **422** for semantically invalid data.
> - **5xx (Server Error)**: The server failed. **500** is the catch-all for unhandled exceptions. **502/503/504** are infrastructure codes typically from reverse proxies.
>
> **In ASP.NET Core**, status codes are set through action result helper methods: `Ok()`, `NotFound()`, `BadRequest()`, `CreatedAtAction()`, `NoContent()`, `Unauthorized()`, `Forbid()`, `Conflict()`, `UnprocessableEntity()`. The `[ApiController]` attribute enables automatic 400 responses for model validation failures. [[Problem Details]] provides a standard JSON format for error responses. The [[Exception Handling]] middleware converts unhandled exceptions to 500 responses.
>
> **The most common mistakes** are: confusing 401 (authentication) with 403 (authorization), returning 200 with error messages in the body, throwing exceptions for validation errors (which become 500s), and returning 404 for empty collections.
>
> **Desktop developer takeaway**: The status code is the HTTP equivalent of your method's return category. 2xx = success, 3xx = "go elsewhere," 4xx = "your fault," 5xx = "my fault." Always check the status code before parsing the response body.
