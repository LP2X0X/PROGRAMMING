---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
---

## Constructor Over-Injection

> [!info] Definition
> **Constructor over-injection** is a code smell where a class requires an excessive number of dependencies (commonly 7 or more), typically indicating that the class violates the Single Responsibility Principle (SRP).

### The Code Smell

```csharp
// OrderController.cs -- TOO MANY DEPENDENCIES
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly ICustomerService _customerService;
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IShippingCalculator _shippingCalculator;
    private readonly ITaxService _taxService;
    private readonly IDiscountEngine _discountEngine;
    private readonly IEmailService _emailService;
    private readonly IAuditLogger _auditLogger;
    private readonly IConfiguration _configuration;

    public OrderController(
        IOrderService orderService,
        ICustomerService customerService,
        IInventoryService inventoryService,
        IPaymentGateway paymentGateway,
        IShippingCalculator shippingCalculator,
        ITaxService taxService,
        IDiscountEngine discountEngine,
        IEmailService emailService,
        IAuditLogger auditLogger,
        IConfiguration configuration)
    {
        _orderService = orderService;
        _customerService = customerService;
        _inventoryService = inventoryService;
        _paymentGateway = paymentGateway;
        _shippingCalculator = shippingCalculator;
        _taxService = taxService;
        _discountEngine = discountEngine;
        _emailService = emailService;
        _auditLogger = auditLogger;
        _configuration = configuration;
    }

    // This class likely handles order creation, pricing, payment, shipping,
    // notifications, and auditing. That is way too many responsibilities.
}
```

### Solution 1: Introduce a Facade Service

Group related dependencies into a higher-level service that encapsulates a coherent set of operations:

```csharp
// IOrderPricingService.cs -- Groups pricing-related concerns
public interface IOrderPricingService
{
    OrderPricing CalculatePricing(Order order, Customer customer);
}

public class OrderPricingService : IOrderPricingService
{
    private readonly IShippingCalculator _shippingCalculator;
    private readonly ITaxService _taxService;
    private readonly IDiscountEngine _discountEngine;

    public OrderPricingService(
        IShippingCalculator shippingCalculator,
        ITaxService taxService,
        IDiscountEngine discountEngine)
    {
        _shippingCalculator = shippingCalculator;
        _taxService = taxService;
        _discountEngine = discountEngine;
    }

    public OrderPricing CalculatePricing(Order order, Customer customer)
    {
        var discount = _discountEngine.Calculate(order, customer);
        var shipping = _shippingCalculator.Calculate(order);
        var tax = _taxService.Calculate(order, customer.Address);
        return new OrderPricing(discount, shipping, tax);
    }
}
```

```csharp
// OrderController.cs -- AFTER REFACTORING
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly ICustomerService _customerService;
    private readonly IOrderPricingService _pricingService;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IEmailService _emailService;

    public OrderController(
        IOrderService orderService,
        ICustomerService customerService,
        IOrderPricingService pricingService,
        IPaymentGateway paymentGateway,
        IEmailService emailService)
    {
        _orderService = orderService;
        _customerService = customerService;
        _pricingService = pricingService;
        _paymentGateway = paymentGateway;
        _emailService = emailService;
    }
}
```

### Solution 2: Use the Mediator Pattern

With a mediator (such as MediatR), the controller delegates to command/query handlers and needs only the mediator itself:

```csharp
// OrderController.cs -- MEDIATOR APPROACH
public class OrderController : ControllerBase
{
    private readonly IMediator _mediator;

    public OrderController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
    {
        var result = await _mediator.Send(new CreateOrderCommand(request));
        return Ok(result);
    }
}
```

```csharp
// CreateOrderCommandHandler.cs -- Each handler has only the dependencies it needs
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, OrderResult>
{
    private readonly IOrderService _orderService;
    private readonly IPaymentGateway _paymentGateway;

    public CreateOrderCommandHandler(
        IOrderService orderService,
        IPaymentGateway paymentGateway)
    {
        _orderService = orderService;
        _paymentGateway = paymentGateway;
    }

    public async Task<OrderResult> Handle(
        CreateOrderCommand command, CancellationToken cancellationToken)
    {
        // Focused handler with only the dependencies it actually uses
        var order = await _orderService.CreateAsync(command.Request);
        await _paymentGateway.ChargeAsync(order.Total);
        return new OrderResult(order.Id);
    }
}
```

> [!tip]
> There is no magic number, but if your constructor has more than 5-7 parameters, stop and ask: "Is this class doing too many things?" Constructor over-injection is a symptom, not the disease. The disease is SRP violation.

> [!summary] Section Summary
> - A constructor with 7+ dependencies is a code smell indicating the class has too many responsibilities
> - Group related dependencies into focused facade/aggregate services to reduce parameter counts
> - The Mediator pattern (MediatR) moves logic into small, focused handlers with minimal dependencies
> - Always treat constructor over-injection as a signal to refactor the class, not just reduce parameter count
> - Splitting the class into smaller, focused classes is often the best solution
