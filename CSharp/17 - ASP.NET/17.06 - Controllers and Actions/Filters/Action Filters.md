---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


Action filters are the **most commonly used** filter type. They run after model binding, wrapping the action method execution itself.

- **Interface**: `IActionFilter` / `IAsyncActionFilter`
- `OnActionExecuting` -- runs after model binding, before the action
- `OnActionExecuted` -- runs after the action method completes
- Can inspect/modify action arguments, modify the result, or short-circuit execution

### Synchronous Implementation

```csharp
public class LogActionFilter : IActionFilter
{
    private readonly ILogger<LogActionFilter> _logger;

    public LogActionFilter(ILogger<LogActionFilter> logger)
    {
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        string actionName = context.ActionDescriptor.DisplayName ?? "Unknown";
        string arguments = JsonSerializer.Serialize(context.ActionArguments);

        _logger.LogInformation(
            "Executing action {Action} with arguments: {Arguments}",
            actionName,
            arguments);
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        string actionName = context.ActionDescriptor.DisplayName ?? "Unknown";

        if (context.Exception is not null)
        {
            _logger.LogError(context.Exception,
                "Action {Action} threw an exception", actionName);
        }
        else
        {
            _logger.LogInformation("Action {Action} executed successfully", actionName);
        }
    }
}
```

### Asynchronous Implementation

```csharp
public class AsyncLogActionFilter : IAsyncActionFilter
{
    private readonly ILogger<AsyncLogActionFilter> _logger;

    public AsyncLogActionFilter(ILogger<AsyncLogActionFilter> logger)
    {
        _logger = logger;
    }

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        string actionName = context.ActionDescriptor.DisplayName ?? "Unknown";

        _logger.LogInformation("Before executing action {Action}", actionName);

        // Call next() to execute the action (and any remaining filters)
        ActionExecutedContext resultContext = await next();

        if (resultContext.Exception is not null)
        {
            _logger.LogError(resultContext.Exception,
                "Action {Action} threw an exception", actionName);
        }
        else
        {
            _logger.LogInformation("After executing action {Action}", actionName);
        }
    }
}
```

```ad-note
The async version uses a single method with an `ActionExecutionDelegate next` parameter. You call `await next()` to execute the action. Everything before `next()` is the "before" logic, everything after is the "after" logic. If you never call `next()`, the action is short-circuited.
```

### Short-Circuiting from an Action Filter

```csharp
public void OnActionExecuting(ActionExecutingContext context)
{
    if (!context.ModelState.IsValid)
    {
        // Short-circuit: skip the action entirely and return a 400
        context.Result = new BadRequestObjectResult(context.ModelState);
    }
}
```
