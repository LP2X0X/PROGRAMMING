---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


When a controller action method declares parameters, ASP.NET Core does not force you to dig through the raw HTTP request manually. Instead, the **model binding** system inspects the incoming request and automatically populates those parameters with values from various parts of the request.

Without model binding, you would write code like this:

```csharp
[HttpGet]
public IActionResult Search()
{
    // Manually extracting values -- tedious and error-prone
    string term = Request.Query["term"];
    int page = int.Parse(Request.Query["page"]);
    string sort = Request.Query["sort"];
    
    // ... use values
}
```

With model binding, this becomes:

```csharp
[HttpGet]
public IActionResult Search(string term, int page, string sort)
{
    // Values are already extracted and converted
    // ... use values directly
}
```

### How It Fits in the Request Pipeline

The model binding system runs **after** routing selects the action method but **before** the action method executes. The sequence is:

1. Routing matches the request to a controller action (see [[17.05 - Routing]])
2. **Model binding** extracts values from the request and populates action parameters
3. **[[Validation]]** runs on the bound model (both data annotations and custom validators)
4. The action method executes with the bound parameters

### What Happens When Binding Fails

If the binder cannot convert a request value to the target type, two things happen:

- The parameter receives its **default value** (`null` for reference types, `0` for `int`, `false` for `bool`, etc.)
- An error is recorded in `ModelState`, which the action can inspect

```csharp
[HttpGet("products/{id}")]
public IActionResult GetProduct(int id)
{
    // If someone requests /products/abc, binding fails:
    // - id will be 0 (default for int)
    // - ModelState.IsValid will be false
    // - ModelState["id"].Errors will contain the conversion error
    
    if (!ModelState.IsValid)
        return BadRequest(ModelState);
    
    // ... proceed with valid id
}
```

```ad-info
With `[ApiController]`, invalid `ModelState` automatically returns a 400 Bad Request response before the action method even runs. You do not need to check `ModelState.IsValid` manually. See [[Validation]] for details.
```
