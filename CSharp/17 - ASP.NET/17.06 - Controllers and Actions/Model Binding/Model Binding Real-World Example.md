---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


This example demonstrates a complete order creation flow that combines multiple binding sources: route values, form data, file uploads, and DI services.

### The Model Classes

```csharp
public class CreateOrderRequest
{
    [Required]
    public int CustomerId { get; set; }
    
    [Required]
    [StringLength(500)]
    public string? Notes { get; set; }
    
    [Required]
    [MinLength(1, ErrorMessage = "At least one item is required.")]
    public List<OrderItemInput> Items { get; set; } = new();
    
    public AddressInput ShippingAddress { get; set; } = new();
}

public class OrderItemInput
{
    [Required]
    public int ProductId { get; set; }
    
    [Range(1, 1000)]
    public int Quantity { get; set; }
}

public class AddressInput
{
    [Required]
    [StringLength(200)]
    public string Street { get; set; } = string.Empty;
    
    [Required]
    [StringLength(100)]
    public string City { get; set; } = string.Empty;
    
    [Required]
    [StringLength(50)]
    public string State { get; set; } = string.Empty;
    
    [Required]
    [RegularExpression(@"^\d{5}(-\d{4})?$")]
    public string ZipCode { get; set; } = string.Empty;
}
```

### The Controller Action

```csharp
[ApiController]
[Route("api/stores/{storeId}/orders")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;
    
    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(
        [FromRoute] int storeId,                     // From URL: /api/stores/7/orders
        [FromBody] CreateOrderRequest request,       // From JSON body
        [FromHeader(Name = "X-Idempotency-Key")] string? idempotencyKey,  // From header
        [FromServices] ILogger<OrdersController> logger)  // From DI
    {
        logger.LogInformation(
            "Creating order for store {StoreId}, customer {CustomerId}, idempotency key: {Key}",
            storeId, request.CustomerId, idempotencyKey);
        
        var order = await _orderService.CreateAsync(storeId, request, idempotencyKey);
        
        return CreatedAtAction(
            nameof(GetOrder),
            new { storeId, orderId = order.Id },
            order);
    }
    
    [HttpGet("{orderId}")]
    public async Task<IActionResult> GetOrder(
        [FromRoute] int storeId,
        [FromRoute] int orderId)
    {
        var order = await _orderService.GetAsync(storeId, orderId);
        return order is null ? NotFound() : Ok(order);
    }
}
```

### Example JSON Request

```json
POST /api/stores/7/orders HTTP/1.1
Content-Type: application/json
X-Idempotency-Key: ord-2024-abc123

{
    "customerId": 42,
    "notes": "Please gift wrap the items",
    "items": [
        { "productId": 101, "quantity": 2 },
        { "productId": 205, "quantity": 1 },
        { "productId": 310, "quantity": 5 }
    ],
    "shippingAddress": {
        "street": "742 Evergreen Terrace",
        "city": "Springfield",
        "state": "IL",
        "zipCode": "62704"
    }
}
```

### Form-Based Order with File Attachment

For a form-based version that includes a file attachment (e.g., a purchase order document):

```csharp
[HttpPost("with-attachment")]
public async Task<IActionResult> CreateOrderWithAttachment(
    [FromRoute] int storeId,
    [FromForm] CreateOrderRequest request,
    [FromForm] IFormFile? purchaseOrderDocument)
{
    if (purchaseOrderDocument is not null)
    {
        if (purchaseOrderDocument.Length > 10 * 1024 * 1024)
            return BadRequest("Purchase order document must be under 10 MB.");
        
        if (purchaseOrderDocument.ContentType != "application/pdf")
            return BadRequest("Only PDF documents are accepted.");
    }
    
    var order = await _orderService.CreateAsync(storeId, request);
    
    if (purchaseOrderDocument is not null)
    {
        using var stream = purchaseOrderDocument.OpenReadStream();
        await _orderService.AttachDocumentAsync(order.Id, stream, purchaseOrderDocument.FileName);
    }
    
    return CreatedAtAction(nameof(GetOrder), new { storeId, orderId = order.Id }, order);
}
```

The corresponding HTML form:

```html
<form method="post"
      action="/api/stores/7/orders/with-attachment"
      enctype="multipart/form-data">
    
    <input type="hidden" name="CustomerId" value="42" />
    <textarea name="Notes" placeholder="Order notes..."></textarea>
    
    <div>
        <input type="number" name="Items[0].ProductId" value="101" />
        <input type="number" name="Items[0].Quantity" value="2" />
    </div>
    <div>
        <input type="number" name="Items[1].ProductId" value="205" />
        <input type="number" name="Items[1].Quantity" value="1" />
    </div>
    
    <input type="text" name="ShippingAddress.Street" placeholder="Street" />
    <input type="text" name="ShippingAddress.City" placeholder="City" />
    <input type="text" name="ShippingAddress.State" placeholder="State" />
    <input type="text" name="ShippingAddress.ZipCode" placeholder="ZIP Code" />
    
    <input type="file" name="PurchaseOrderDocument" accept=".pdf" />
    
    <button type="submit">Place Order</button>
</form>
```
