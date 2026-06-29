---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


- [[Exception Handling]] -- global exception handling that feeds into ProblemDetails responses
- [[ILogger and Logging]] -- structured logging for the server-side details behind each ProblemDetails traceId
- [[Middleware Overview]] -- how `UseExceptionHandler` and `UseStatusCodePages` fit in the pipeline
- [[Routing Overview]] -- how routing affects which endpoints produce ProblemDetails

---


- [[API Versioning]] -- maintaining ProblemDetails consistency across API versions
- [[OpenAPI and Swagger]] -- documenting ProblemDetails responses in your API specification
- [[Content Negotiation]] -- how the framework decides between JSON and XML ProblemDetails
- [[Health Checks]] -- health check endpoints and their relationship to error monitoring

---


> [!tip] Complete Summary
> **ProblemDetails** (RFC 9457, formerly RFC 7807) is the industry-standard format for HTTP API error responses. It provides five standard members: `type` (URI identifying the problem), `title` (generic summary), `status` (HTTP code), `detail` (occurrence-specific explanation), and `instance` (request reference). Custom extension members like `errorCode`, `traceId`, and `timestamp` add context without breaking the standard.
>
> **ASP.NET Core's built-in support** includes the `ProblemDetails` and `ValidationProblemDetails` classes, the `[ApiController]` attribute (which automatically returns ProblemDetails for errors and ValidationProblemDetails for validation failures), and `AddProblemDetails()` (.NET 7+) for global configuration.
>
> **`AddProblemDetails()`** with `CustomizeProblemDetails` is the single best place to add cross-cutting metadata (trace IDs, timestamps, node identifiers) to every error response. It applies to all ProblemDetails from any source: exception handlers, status code pages, `[ApiController]`, and manual creation.
>
> **`IProblemDetailsService`** enables ProblemDetails generation from middleware, minimal APIs, and anywhere outside controllers, while respecting global customizations. In minimal APIs, `Results.Problem()` and `Results.ValidationProblem()` provide convenient helpers.
>
> The most powerful pattern combines **`IExceptionHandler`** (.NET 8+) with `IProblemDetailsService`: domain exceptions (NotFoundException, ValidationException, etc.) are caught by specific handlers and converted to typed ProblemDetails with appropriate `type` URIs, status codes, and `errorCode` extensions. A fallback handler catches everything else and returns a generic 500 with no internal details.
>
> **Two audiences** are served simultaneously: machines use `type`, `status`, and `errorCode` for programmatic handling; humans read `title`, `detail`, and look up `traceId` in server logs. Keep `title` generic (same for every occurrence of the problem type) and `detail` specific (describes this particular failure).
>
> **Validation errors** use `ValidationProblemDetails` with an `errors` dictionary mapping field names to messages. `[ApiController]` generates this automatically for invalid model state before the action method runs. Customize via `ApiBehaviorOptions.InvalidModelStateResponseFactory`.
>
> The result is a consistent, documented, machine-readable error format across every endpoint in your API -- enabling generic client-side error handling and efficient debugging through trace ID correlation with [[ILogger and Logging|server-side logs]].
