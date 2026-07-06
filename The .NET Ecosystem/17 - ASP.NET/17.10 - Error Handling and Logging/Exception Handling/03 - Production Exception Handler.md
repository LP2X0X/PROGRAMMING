---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


For production, ASP.NET Core provides `UseExceptionHandler()`, which catches exceptions from downstream middleware, logs them, and re-executes the pipeline to a specified error-handling path.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    // Catches any unhandled exception from downstream middleware,
    // then re-executes the pipeline targeting "/Home/Error"
    app.UseExceptionHandler("/Home/Error");
}

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

## How UseExceptionHandler Works Internally

Understanding the internal mechanism is essential for debugging. Here is what happens step by step:

1. An exception is thrown somewhere in the pipeline (controller, service, another middleware)
2. The exception propagates up the middleware chain until it reaches `UseExceptionHandler`
3. `UseExceptionHandler` **catches** the exception and **clears** the response (status code, headers, body)
4. It sets the response status code to **500**
5. It stores the exception details in `IExceptionHandlerPathFeature` on the `HttpContext.Features` collection
6. It **re-executes** the middleware pipeline from the exception handler middleware onward, but with `HttpContext.Request.Path` set to the error path (e.g., `/Home/Error`)
7. The re-executed pipeline hits the error controller/endpoint, which renders the error page
8. If the error handler itself throws, the middleware catches that too and returns a plain-text response

> [!ad-note] Why Re-Execute Instead of Writing the Error Directly?
> When your code throws, the response may already be ==partially written== — some headers may have been sent, some data may have been flushed to the response body. At that point, the response is in a ==corrupted state== and you cannot reliably write a clean JSON or HTML error body on top of it. By **clearing the response** (step 3) and **re-executing the pipeline with a new path** (step 6), the exception handler creates a ==clean slate==. The second pass goes through the full pipeline — DI, serialization, content negotiation — and produces a proper, well-formatted error response as if it were a normal request.

```csharp
// The error controller that handles re-executed requests
public class HomeController : Controller
{
    [Route("/Home/Error")]
    [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
    public IActionResult Error()
    {
        // Access the original exception details
        var exceptionFeature = HttpContext.Features
            .Get<IExceptionHandlerPathFeature>();

        // Log or inspect the exception
        var originalException = exceptionFeature?.Error;
        var originalPath = exceptionFeature?.Path;

        return View(new ErrorViewModel
        {
            RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier,
            Message = "An unexpected error occurred. Please try again later."
        });
    }
}
```

> [!warning] Common Misconception
> `UseExceptionHandler` does **not** redirect the browser to the error path. It re-executes the pipeline internally on the same request. The client's URL does not change, there is no 302 redirect, and the original HTTP method is preserved. This means if a POST request fails, the error handler receives the context of a POST request, not a GET.

There is also an overload that accepts a lambda for inline error handling without a separate error path:

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        context.Response.StatusCode = 500;
        context.Response.ContentType = "text/html";

        var exceptionFeature = context.Features
            .Get<IExceptionHandlerPathFeature>();

        await context.Response.WriteAsync("<h1>An error occurred</h1>");
        // Do NOT write exception details in production
    });
});
```

> [!summary] Section Summary
> - `UseExceptionHandler` catches unhandled exceptions and re-executes the pipeline to a specified error path
> - The re-execution is internal -- no browser redirect, no URL change, same HTTP method
> - The original exception is accessible via `IExceptionHandlerPathFeature` on `HttpContext.Features`
> - Always place `UseExceptionHandler` as the outermost middleware to catch all downstream exceptions
> - A lambda overload allows inline error handling without a separate controller action
