---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---

> [!abstract] Overview
> When an API returns an error, what should the response body look like? Without a standard, every API invents its own format -- `{ "error": "..." }`, `{ "message": "...", "code": 123 }`, `{ "errors": [...] }` -- and every client must be taught each API's specific error shape. This is the problem that **ProblemDetails** solves.
>
> **ProblemDetails** is a standardized error response format defined by **RFC 9457** (formerly RFC 7807), titled "Problem Details for HTTP APIs." It provides a machine-readable, extensible JSON structure for communicating errors from HTTP APIs. ASP.NET Core has built-in support for generating ProblemDetails responses, making it straightforward to adopt this standard across all your API endpoints.


RFC 9457 defines a standard way for HTTP APIs to communicate errors to clients. The key motivation is **consistency**: if every API uses the same error format, client libraries can handle errors generically instead of needing custom parsing logic per API.

The standard is:
- **Media type**: `application/problem+json` (or `application/problem+xml`)
- **Not tied to any framework** -- it is a pure HTTP/JSON standard used by APIs written in any language
- **Extensible** -- you can add custom members beyond the standard fields
- **Machine-readable** -- the `type` field is a URI that identifies the error type programmatically

## History

- **RFC 7807** (2016): Original specification by M. Nottingham and E. Wilde
- **RFC 9457** (2023): Supersedes RFC 7807 with clarifications and minor updates
- ASP.NET Core adopted ProblemDetails support starting in ASP.NET Core 2.1, with significant improvements in .NET 7 and .NET 8

> [!info] Definition
> **ProblemDetails** = a standardized JSON (or XML) format for conveying machine-readable details of errors in an HTTP response, as defined by RFC 9457. It provides a consistent shape that any HTTP client can parse, regardless of which API generated the error.

> [!summary] Section Summary
> - ProblemDetails (RFC 9457, formerly RFC 7807) is the industry standard for HTTP API error responses
> - It provides a consistent, machine-readable JSON format with well-defined fields
> - The media type `application/problem+json` signals that the response body is a ProblemDetails object
> - Adopted by ASP.NET Core since version 2.1, with full support in .NET 7/8
