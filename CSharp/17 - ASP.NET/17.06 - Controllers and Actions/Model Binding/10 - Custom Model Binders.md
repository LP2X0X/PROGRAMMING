---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


When the built-in binders cannot handle your scenario, you can implement a custom model binder by implementing the `IModelBinder` interface.

### Use Case: Comma-Separated Values to List

A common need is binding a comma-separated query string value like `?ids=1,2,3` into a `List<int>`:

```csharp
using Microsoft.AspNetCore.Mvc.ModelBinding;

public class CommaSeparatedModelBinder : IModelBinder
{
    public Task BindModelAsync(ModelBindingContext bindingContext)
    {
        ArgumentNullException.ThrowIfNull(bindingContext);
        
        var modelName = bindingContext.ModelName;
        var valueProviderResult = bindingContext.ValueProvider.GetValue(modelName);
        
        if (valueProviderResult == ValueProviderResult.None)
            return Task.CompletedTask;
        
        bindingContext.ModelState.SetModelValue(modelName, valueProviderResult);
        
        var value = valueProviderResult.FirstValue;
        
        if (string.IsNullOrWhiteSpace(value))
            return Task.CompletedTask;
        
        // Split the comma-separated string and parse each element
        var items = new List<int>();
        foreach (var segment in value.Split(',', StringSplitOptions.RemoveEmptyEntries))
        {
            if (int.TryParse(segment.Trim(), out int parsed))
            {
                items.Add(parsed);
            }
            else
            {
                bindingContext.ModelState.TryAddModelError(
                    modelName,
                    $"'{segment.Trim()}' is not a valid integer.");
                return Task.CompletedTask;
            }
        }
        
        bindingContext.Result = ModelBindingResult.Success(items);
        return Task.CompletedTask;
    }
}
```

### Applying the Custom Binder

There are several ways to register a custom binder.

**Per-parameter with `[ModelBinder]`:**

```csharp
[HttpGet("products")]
public IActionResult GetByIds(
    [ModelBinder(BinderType = typeof(CommaSeparatedModelBinder))] List<int> ids)
{
    // GET /products?ids=1,2,3
    // ids = [1, 2, 3]
}
```

**Via a custom attribute:**

```csharp
[AttributeUsage(AttributeTargets.Parameter | AttributeTargets.Property)]
public class CommaSeparatedAttribute : ModelBinderAttribute
{
    public CommaSeparatedAttribute()
        : base(typeof(CommaSeparatedModelBinder))
    {
    }
}

// Usage becomes cleaner:
[HttpGet("products")]
public IActionResult GetByIds([CommaSeparated] List<int> ids)
{
    // GET /products?ids=1,2,3
}
```

**Globally via a model binder provider:**

```csharp
public class CommaSeparatedModelBinderProvider : IModelBinderProvider
{
    public IModelBinder? GetBinder(ModelBinderProviderContext context)
    {
        ArgumentNullException.ThrowIfNull(context);
        
        // Apply this binder to any List<int> parameter decorated with [CommaSeparated]
        if (context.Metadata.ModelType == typeof(List<int>)
            && context.Metadata is Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelMetadata metadata
            && metadata.Attributes.ParameterAttributes?.OfType<CommaSeparatedAttribute>().Any() == true)
        {
            return new CommaSeparatedModelBinder();
        }
        
        return null;
    }
}

// Register in Program.cs:
builder.Services.AddControllers(options =>
{
    options.ModelBinderProviders.Insert(0, new CommaSeparatedModelBinderProvider());
});
```

```ad-note
Custom binder providers are checked in order. Insert your provider at position 0 to ensure it is evaluated before the built-in providers. If your provider returns `null`, the next provider in the list is tried.
```
