---
tags:
  - csharp
  - asp-net-core
  - action-results
  - controllers
---


Below is a complete controller demonstrating all the common action result patterns for a typical CRUD API. This uses `ActionResult<T>` throughout.

```csharp
using Microsoft.AspNetCore.Mvc;

namespace MyStore.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;

    public OrdersController(
        IOrderRepository orderRepository,
        IProductRepository productRepository)
    {
        _orderRepository = orderRepository;
        _productRepository = productRepository;
    }

    // -------------------------------------------------------
    // GET api/orders
    // Returns: 200 Ok with list of orders
    // -------------------------------------------------------
    [HttpGet]
    public async Task<ActionResult<IEnumerable<OrderSummaryDto>>> GetOrders(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20)
    {
        var orders = await _orderRepository.GetPagedAsync(page, pageSize);
        return Ok(orders);
    }

    // -------------------------------------------------------
    // GET api/orders/{id}
    // Returns: 200 Ok with order, or 404 Not Found
    // -------------------------------------------------------
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<OrderDetailDto>> GetOrder(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        return order;
    }

    // -------------------------------------------------------
    // POST api/orders
    // Returns: 201 Created with location header, or 400 Bad Request
    // -------------------------------------------------------
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<OrderDetailDto>> CreateOrder(
        CreateOrderRequest request)
    {
        // [ApiController] handles model state validation automatically.
        // If we reach here, model state is valid.

        // Business validation
        foreach (var item in request.Items)
        {
            var product = await _productRepository.GetByIdAsync(item.ProductId);
            if (product is null)
            {
                ModelState.AddModelError(
                    nameof(item.ProductId),
                    $"Product {item.ProductId} does not exist.");

                return BadRequest(ModelState);
            }
        }

        var order = await _orderRepository.CreateAsync(request);

        // Returns 201 with a Location header like:
        //   Location: https://mystore.com/api/orders/42
        return CreatedAtAction(
            actionName: nameof(GetOrder),
            routeValues: new { id = order.Id },
            value: order);
    }

    // -------------------------------------------------------
    // PUT api/orders/{id}
    // Returns: 204 No Content, 404 Not Found, or 409 Conflict
    // -------------------------------------------------------
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status409Conflict)]
    public async Task<IActionResult> UpdateOrder(
        int id,
        UpdateOrderRequest request)
    {
        var existingOrder = await _orderRepository.GetByIdAsync(id);

        if (existingOrder is null)
            return NotFound();

        // Concurrency check using a version/timestamp
        if (existingOrder.Version != request.ExpectedVersion)
        {
            return Conflict(new
            {
                Message = "The order was modified by another user.",
                CurrentVersion = existingOrder.Version
            });
        }

        await _orderRepository.UpdateAsync(id, request);

        return NoContent();
    }

    // -------------------------------------------------------
    // DELETE api/orders/{id}
    // Returns: 204 No Content, or 404 Not Found
    // -------------------------------------------------------
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteOrder(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        await _orderRepository.DeleteAsync(id);

        return NoContent();
    }

    // -------------------------------------------------------
    // POST api/orders/{id}/cancel
    // Returns: 200 Ok, 404 Not Found, or 422 Unprocessable Entity
    // -------------------------------------------------------
    [HttpPost("{id}/cancel")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status422UnprocessableEntity)]
    public async Task<ActionResult<OrderDetailDto>> CancelOrder(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        if (order.Status == OrderStatus.Shipped)
        {
            return UnprocessableEntity(new
            {
                Message = "Cannot cancel an order that has already shipped."
            });
        }

        var cancelledOrder = await _orderRepository.CancelAsync(id);

        return Ok(cancelledOrder);
    }

    // -------------------------------------------------------
    // GET api/orders/{id}/invoice
    // Returns: 200 Ok (PDF file), or 404 Not Found
    // -------------------------------------------------------
    [HttpGet("{id}/invoice")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DownloadInvoice(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        byte[] pdfBytes = await _orderRepository.GenerateInvoicePdfAsync(id);

        return File(
            pdfBytes,
            "application/pdf",
            $"invoice-{id}.pdf");
    }

    // -------------------------------------------------------
    // POST api/orders/{id}/process
    // Returns: 202 Accepted (long-running operation)
    // -------------------------------------------------------
    [HttpPost("{id}/process")]
    [ProducesResponseType(StatusCodes.Status202Accepted)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> ProcessOrder(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        // Queue for background processing instead of doing it synchronously
        await _orderRepository.QueueForProcessingAsync(id);

        return Accepted(
            uri: Url.Action(nameof(GetOrder), new { id }),
            value: new { Message = "Order processing has been queued.", OrderId = id });
    }
}
```

```ad-summary
**Pattern recap for the OrdersController above:**
- **GET collection** -- `Ok(list)` always (200)
- **GET single** -- `Ok(item)` or `NotFound()` (200 / 404)
- **POST create** -- `CreatedAtAction(...)` or `BadRequest(modelState)` (201 / 400)
- **PUT update** -- `NoContent()`, `NotFound()`, or `Conflict(...)` (204 / 404 / 409)
- **DELETE** -- `NoContent()` or `NotFound()` (204 / 404)
- **Custom action** -- `Ok(result)`, `NotFound()`, or `UnprocessableEntity(...)` (200 / 404 / 422)
- **File download** -- `File(bytes, contentType, fileName)` or `NotFound()` (200 / 404)
- **Long-running** -- `Accepted(uri, value)` or `NotFound()` (202 / 404)

Note that PUT and DELETE return `IActionResult` instead of `ActionResult<T>` because their success path (204 No Content) has no body to type.
```
