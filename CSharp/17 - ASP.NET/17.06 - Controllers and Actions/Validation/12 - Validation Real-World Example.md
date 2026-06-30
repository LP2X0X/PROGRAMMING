---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


Putting it all together: a `CreateOrderRequest` model with comprehensive validation using Data Annotations, `IValidatableObject`, and the corresponding controller action.

### The Model

```csharp
using System.ComponentModel.DataAnnotations;

public class CreateOrderRequest : IValidatableObject
{
    [Required(ErrorMessage = "Customer name is required.")]
    [StringLength(200, MinimumLength = 2,
        ErrorMessage = "Customer name must be between 2 and 200 characters.")]
    [Display(Name = "Customer Name")]
    public string CustomerName { get; set; } = string.Empty;

    [Required(ErrorMessage = "Email address is required.")]
    [EmailAddress(ErrorMessage = "Please provide a valid email address.")]
    public string Email { get; set; } = string.Empty;

    [Required]
    [Phone]
    [Display(Name = "Phone Number")]
    public string PhoneNumber { get; set; } = string.Empty;

    [Required(ErrorMessage = "Product ID is required.")]
    public Guid? ProductId { get; set; }

    [Required]
    [Range(1, 10000, ErrorMessage = "Quantity must be between 1 and 10,000.")]
    public int? Quantity { get; set; }

    [Required]
    [StringLength(500)]
    [Display(Name = "Shipping Address")]
    public string ShippingAddress { get; set; } = string.Empty;

    [StringLength(1000)]
    [Display(Name = "Special Instructions")]
    public string? SpecialInstructions { get; set; }

    public bool IsGiftOrder { get; set; }

    [StringLength(200)]
    [Display(Name = "Gift Message")]
    public string? GiftMessage { get; set; }

    [Required]
    [AllowedValues("Standard", "Express", "Overnight",
        ErrorMessage = "Shipping method must be Standard, Express, or Overnight.")]
    [Display(Name = "Shipping Method")]
    public string ShippingMethod { get; set; } = string.Empty;

    [Required]
    public DateTime? RequestedDeliveryDate { get; set; }

    // Cross-property validation
    public IEnumerable<ValidationResult> Validate(
        ValidationContext validationContext)
    {
        // Gift message required for gift orders
        if (IsGiftOrder && string.IsNullOrWhiteSpace(GiftMessage))
        {
            yield return new ValidationResult(
                "A gift message is required for gift orders.",
                new[] { nameof(GiftMessage) });
        }

        // Delivery date must be in the future
        if (RequestedDeliveryDate.HasValue
            && RequestedDeliveryDate.Value.Date <= DateTime.UtcNow.Date)
        {
            yield return new ValidationResult(
                "Requested delivery date must be in the future.",
                new[] { nameof(RequestedDeliveryDate) });
        }

        // Express/Overnight requires at least 1 day lead time,
        // Standard requires at least 3 days
        if (RequestedDeliveryDate.HasValue)
        {
            int minimumLeadDays = ShippingMethod switch
            {
                "Overnight" => 1,
                "Express" => 2,
                "Standard" => 3,
                _ => 3
            };

            var earliestDelivery = DateTime.UtcNow.Date.AddDays(minimumLeadDays);

            if (RequestedDeliveryDate.Value.Date < earliestDelivery)
            {
                yield return new ValidationResult(
                    $"{ShippingMethod} shipping requires at least " +
                    $"{minimumLeadDays} business day(s) lead time.",
                    new[] { nameof(RequestedDeliveryDate) });
            }
        }
    }
}
```

### The Controller Action

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly IProductService _productService;

    public OrdersController(
        IOrderService orderService,
        IProductService productService)
    {
        _orderService = orderService;
        _productService = productService;
    }

    [HttpPost]
    [ProducesResponseType(typeof(OrderResponse), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ValidationProblemDetails),
        StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> Create(CreateOrderRequest request)
    {
        // Data Annotations + IValidatableObject already validated by [ApiController].
        // Add business-rule validation that requires service calls.

        var product = await _productService.GetByIdAsync(request.ProductId!.Value);

        if (product is null)
        {
            ModelState.AddModelError(nameof(request.ProductId),
                "The specified product does not exist.");
            return ValidationProblem();
        }

        if (product.Stock < request.Quantity!.Value)
        {
            ModelState.AddModelError(nameof(request.Quantity),
                $"Insufficient stock. Only {product.Stock} units available.");
            return ValidationProblem();
        }

        var order = await _orderService.CreateAsync(request);

        return CreatedAtAction(
            nameof(GetById),
            new { id = order.Id },
            OrderResponse.FromEntity(order));
    }

    [HttpGet("{id:guid}")]
    public async Task<IActionResult> GetById(Guid id)
    {
        var order = await _orderService.GetByIdAsync(id);
        return order is null
            ? NotFound()
            : Ok(OrderResponse.FromEntity(order));
    }
}
```

### Example Validation Error Responses

**Missing required fields:**

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "CustomerName": ["Customer name is required."],
    "Email": ["Email address is required."],
    "ProductId": ["The ProductId field is required."],
    "Quantity": ["The Quantity field is required."],
    "ShippingAddress": ["The ShippingAddress field is required."],
    "ShippingMethod": ["Shipping method must be Standard, Express, or Overnight."],
    "RequestedDeliveryDate": ["The RequestedDeliveryDate field is required."]
  },
  "traceId": "00-a1b2c3d4e5f6-789abc-00"
}
```

**Cross-property and business-rule errors:**

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "GiftMessage": ["A gift message is required for gift orders."],
    "RequestedDeliveryDate": [
      "Standard shipping requires at least 3 business day(s) lead time."
    ],
    "Quantity": ["Insufficient stock. Only 5 units available."]
  },
  "traceId": "00-f6e5d4c3b2a1-def012-00"
}
```

```ad-summary
A real-world model combines `[Required]`, `[StringLength]`, `[Range]`, `[EmailAddress]`, `[AllowedValues]`, and other Data Annotations for property-level rules. `IValidatableObject` handles cross-property rules (gift message requirement, delivery lead time). The `[ApiController]` attribute handles the annotation-level validation automatically, while business-rule errors (stock check, product existence) are added to `ModelState` in the action method.
```
