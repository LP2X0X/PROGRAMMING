---
tags: [csharp, asp-net-core, http, status-codes, web-api, fundamentals]
---


**Success responses** mean the server received the request, understood it, and processed it successfully. These are the "happy path" codes.

| Code | Name | Meaning | ASP.NET Core Helper |
|---|---|---|---|
| **200** | OK | Standard success response. Body contains the requested data. | `Ok()` / `Ok(value)` |
| **201** | Created | A new resource was successfully created. Should include a `Location` header pointing to the new resource. | `Created()` / `CreatedAtAction()` / `CreatedAtRoute()` |
| **202** | Accepted | Request was accepted for processing, but processing is not complete. Used for async/queued operations. | `Accepted()` |
| **204** | No Content | Success, but there is no body to return. Common for DELETE and PUT operations. | `NoContent()` |

#### 200 OK -- The Default Success

The most common status code. The request succeeded and the response body contains data.

```csharp
[HttpGet]
public async Task<IActionResult> GetAll()
{
    var products = await _repository.GetAllAsync();
    return Ok(products); // 200 + JSON array
}
```

#### 201 Created -- Resource Was Created

Used after a successful POST that creates a new resource. Best practice is to include a `Location` header that points to the newly created resource's URL.

```csharp
[HttpPost]
public async Task<IActionResult> Create(CreateProductDto dto)
{
    var product = await _service.CreateAsync(dto);

    // Returns 201 with Location header pointing to GetById endpoint
    return CreatedAtAction(
        actionName: nameof(GetById),
        routeValues: new { id = product.Id },
        value: product
    );
}
```

The response will look like:

```
HTTP/1.1 201 Created
Location: /api/products/42
Content-Type: application/json

{"id": 42, "name": "New Widget", "price": 19.99}
```

#### 204 No Content -- Success, Nothing to Return

Used when the operation succeeds but there is nothing meaningful to return in the body. Common for `DELETE` and `PUT` operations.

```csharp
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(int id)
{
    var existed = await _repository.DeleteAsync(id);
    if (!existed)
        return NotFound();

    return NoContent(); // 204 -- deleted successfully, nothing to return
}

[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, UpdateProductDto dto)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    await _service.UpdateAsync(id, dto);
    return NoContent(); // 204 -- updated successfully
}
```

> [!ad-warning] Common Misconception: Always Return 200
> Beginners often return `200 OK` for everything -- including creates and deletes. This makes the API less self-describing. Use `201 Created` when something was created and `204 No Content` when there is nothing to return. Your API consumers (and tools like Swagger) benefit from the precision.

> [!ad-tip] 202 Accepted -- For Async Operations
> If your API kicks off a long-running background job (e.g., generating a report, processing an upload), return `202 Accepted` to indicate "I received your request and will process it, but it is not done yet." Optionally include a URL where the client can poll for status.
> ```csharp
> [HttpPost("reports")]
> public IActionResult GenerateReport(ReportRequest request)
> {
>     var jobId = _backgroundJobService.Enqueue(request);
>     return Accepted(
>         uri: $"/api/reports/status/{jobId}",
>         value: new { jobId, status = "processing" }
>     );
> }
> ```

> [!summary] Section Summary
> - 200 OK: standard success with data in the body
> - 201 Created: resource was created; include a `Location` header pointing to it
> - 202 Accepted: request accepted for async processing; not yet complete
> - 204 No Content: success but no body to return (DELETE, PUT)
> - Use the precise code, not 200 for everything -- it makes your API self-describing
