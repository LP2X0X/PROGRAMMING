---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - fundamentals
---

## The Problem DI Solves

Without DI, classes create their own dependencies using `new`. This creates **tight coupling** -- the class is permanently bound to specific concrete implementations, making it difficult to test, extend, or change behavior.

### Before: Tight Coupling Without DI

```csharp
public class OrderService
{
    private readonly SqlInventoryRepository _inventory;
    private readonly StripePaymentGateway _paymentGateway;
    private readonly SmtpEmailService _emailService;

    public OrderService()
    {
        // This class decides exactly which implementations to use
        _inventory = new SqlInventoryRepository("Server=prod;Database=Inventory;...");
        _paymentGateway = new StripePaymentGateway("sk_live_abc123");
        _emailService = new SmtpEmailService("smtp.company.com", 587);
    }

    public OrderResult PlaceOrder(Order order)
    {
        bool inStock = _inventory.CheckStock(order.ProductId, order.Quantity);
        if (!inStock)
            return OrderResult.OutOfStock;

        PaymentResult payment = _paymentGateway.Charge(order.Total, order.PaymentInfo);
        if (!payment.Success)
            return OrderResult.PaymentFailed;

        _inventory.ReserveStock(order.ProductId, order.Quantity);
        _emailService.SendConfirmation(order.CustomerEmail, order);

        return OrderResult.Success;
    }
}
```

> [!danger] Problems With This Approach
> - **Untestable**: You cannot unit test `OrderService` without hitting a real database, Stripe API, and SMTP server.
> - **Rigid**: Switching from Stripe to PayPal requires modifying `OrderService` source code.
> - **Hidden dependencies**: Anyone reading the class signature has no idea what it needs to function.
> - **Connection strings and secrets are hardcoded** inside the business logic class.
> - **Violation of Single Responsibility**: `OrderService` is responsible for both its business logic and for deciding which implementations to use.

### After: Loose Coupling With DI

```csharp
public class OrderService : IOrderService
{
    private readonly IInventoryRepository _inventory;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IEmailService _emailService;

    // Dependencies are declared as interfaces and injected through the constructor
    public OrderService(
        IInventoryRepository inventory,
        IPaymentGateway paymentGateway,
        IEmailService emailService)
    {
        _inventory = inventory;
        _paymentGateway = paymentGateway;
        _emailService = emailService;
    }

    public OrderResult PlaceOrder(Order order)
    {
        bool inStock = _inventory.CheckStock(order.ProductId, order.Quantity);
        if (!inStock)
            return OrderResult.OutOfStock;

        PaymentResult payment = _paymentGateway.Charge(order.Total, order.PaymentInfo);
        if (!payment.Success)
            return OrderResult.PaymentFailed;

        _inventory.ReserveStock(order.ProductId, order.Quantity);
        _emailService.SendConfirmation(order.CustomerEmail, order);

        return OrderResult.Success;
    }
}
```

> [!tip] What Changed
> The business logic in `PlaceOrder` is **identical**. The only difference is how `OrderService` gets its dependencies. Now you can:
> - **Unit test** by passing in mock implementations of `IInventoryRepository`, `IPaymentGateway`, and `IEmailService`.
> - **Swap implementations** (e.g., `StripePaymentGateway` to `PayPalPaymentGateway`) without touching `OrderService`.
> - **See all dependencies** just by reading the constructor signature.

> [!example] Unit Testing With DI
> ```csharp
> [Fact]
> public void PlaceOrder_WhenOutOfStock_ReturnsOutOfStock()
> {
>     // Arrange -- using mock implementations
>     var mockInventory = new Mock<IInventoryRepository>();
>     mockInventory
>         .Setup(i => i.CheckStock(It.IsAny<int>(), It.IsAny<int>()))
>         .Returns(false);
> 
>     var mockPayment = new Mock<IPaymentGateway>();
>     var mockEmail = new Mock<IEmailService>();
> 
>     var service = new OrderService(
>         mockInventory.Object,
>         mockPayment.Object,
>         mockEmail.Object);
> 
>     // Act
>     var result = service.PlaceOrder(new Order { ProductId = 1, Quantity = 100 });
> 
>     // Assert
>     Assert.Equal(OrderResult.OutOfStock, result);
>     mockPayment.Verify(p => p.Charge(It.IsAny<decimal>(), It.IsAny<PaymentInfo>()), Times.Never);
> }
> ```

> [!summary] Section Summary
> - Without DI, classes use `new` to create dependencies, leading to tight coupling between high-level and low-level modules.
> - Tightly coupled code is difficult to test because you cannot substitute real services with mocks or fakes.
> - DI solves this by depending on interfaces and receiving concrete implementations from the outside.
> - The business logic itself does not change -- only how dependencies are obtained changes.
> - With DI, unit testing becomes straightforward because you can inject mock implementations.
