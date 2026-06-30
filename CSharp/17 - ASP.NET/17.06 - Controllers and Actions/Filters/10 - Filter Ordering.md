---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


### Order by Filter Type

Filters always execute in type order regardless of registration:

1. Authorization
2. Resource
3. Action
4. Exception
5. Result

### Order Within the Same Type

Within the same filter type, the default execution order is:

1. **Global** filters (registered in `AddControllers`)
2. **Controller-level** filters (attribute on the controller class)
3. **Action-level** filters (attribute on the action method)

The "after" hooks run in **reverse** (action -> controller -> global).

### Custom Ordering with IOrderedFilter

Override the default scope-based ordering by implementing `IOrderedFilter`:

```csharp
public class AuditLogFilter : ActionFilterAttribute, IOrderedFilter
{
    // Lower values execute first. Default is 0.
    public new int Order { get; set; } = -1; // Run before other action filters
}

public class ValidationFilter : ActionFilterAttribute, IOrderedFilter
{
    public new int Order { get; set; } = 0;
}

public class CachingFilter : ActionFilterAttribute, IOrderedFilter
{
    public new int Order { get; set; } = 1; // Run after audit and validation
}
```

```csharp
// Or set order when registering globally
builder.Services.AddControllers(options =>
{
    options.Filters.Add<AuditLogFilter>(order: -1);
    options.Filters.Add<ValidationFilter>(order: 0);
    options.Filters.Add<CachingFilter>(order: 1);
});
```

```ad-note
When two filters have the **same** `Order` value, the scope-based rule applies (global before controller before action). Lower `Order` values always run their "before" hook first, and their "after" hook last.
```
