---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---

> [!abstract] Overview
> Every web application will eventually encounter unexpected errors -- database timeouts, null references, failed external API calls, file system issues. How you handle these errors determines whether your users see a helpful message or a raw stack trace, and whether your operations team can diagnose problems or is left guessing.
>
> In ASP.NET Core, exception handling is built around the **middleware pipeline**. The framework provides built-in middleware for development and production error handling, and gives you extension points to build custom solutions that handle both MVC views and API JSON responses from a single codebase.


When an unhandled exception escapes your application code and reaches the ASP.NET Core hosting layer, the default behavior is to return a **blank 500 Internal Server Error** response with no body. This is terrible for everyone:

- **Users** see a cryptic error or a blank page with no guidance
- **Developers** have no diagnostic information in the response (unless they check server logs)
- **Attackers** may receive stack traces, connection strings, or internal paths if the Developer Exception Page is accidentally enabled in production

The consequences of poor exception handling are real:

1. **Information leakage** -- stack traces reveal internal implementation details, file paths, assembly names, database connection strings, and even query parameters. This is a security vulnerability.
2. **Poor user experience** -- users who see a raw 500 error have no idea what happened or what to do next
3. **Difficult debugging** -- without structured logging of exceptions, diagnosing production issues becomes guesswork
4. **Inconsistent responses** -- APIs that sometimes return JSON and sometimes return HTML error pages break client applications

> [!danger]
> An unhandled exception in production without proper error handling middleware can expose your application's internals. The default Kestrel response for an unhandled exception is a 500 status with an empty body, but if the Developer Exception Page is accidentally left on, the full stack trace, source code snippets, request headers, cookies, and routing data are all visible to anyone making the request.

> [!summary] Section Summary
> - Unhandled exceptions produce blank 500 responses by default, or leak internal details if the Developer Exception Page is on in production
> - Poor error handling leads to security vulnerabilities, bad UX, difficult debugging, and inconsistent API responses
> - Proper exception handling middleware is not optional -- it is a fundamental requirement for any production application
