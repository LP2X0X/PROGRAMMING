---
tags:
  - csharp
  - asp-net-core
  - web-api
  - conventions
  - openapi
---


**API conventions** are ==standardized patterns that ensure consistent behavior, predictable responses, and clear documentation== across all endpoints in a Web API. Rather than leaving each developer to decide individually how errors are returned, what status codes map to what scenarios, or how API documentation is generated, conventions establish a uniform contract that both producers and consumers of the API can rely on.

API conventions in ASP.NET Core address several concerns:

- **Response type documentation** -- what status codes and response bodies each action can return
- **Error response format** -- a standardized structure for error details (ProblemDetails)
- **API documentation** -- auto-generated OpenAPI/Swagger specs from code metadata
- **Versioning** -- how breaking changes are introduced without disrupting existing consumers
- **Discoverability** -- making APIs self-describing via hypermedia links (HATEOAS)

Without conventions, APIs become inconsistent: one endpoint returns `{ "error": "Not found" }`, another returns `{ "message": "Resource missing", "code": 404 }`, and a third returns a bare string. Conventions eliminate this chaos.

> [!ad-note]
> Conventions are not just about documentation. They influence middleware behavior, exception handling, content negotiation, and how tools like Swagger UI render your API. See [[Content Negotiation]] for how response formatting interacts with these conventions.

> [!summary] Section Summary
> API conventions establish uniform patterns for response types, error formats, documentation, versioning, and discoverability across all endpoints in a Web API.
