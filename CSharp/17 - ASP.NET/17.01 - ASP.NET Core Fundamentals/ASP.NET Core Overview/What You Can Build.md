---
tags: [csharp, asp-net-core, fundamentals, web]
---


ASP.NET Core is not a single-purpose framework. It supports multiple application models under one umbrella.

| Application Model | What It Does |
|---|---|
| **[[Minimal APIs]]** | Simple HTTP APIs that can be consumed by mobile applications or browser-based single-page applications (SPAs) |
| **[[API Controllers\|Web APIs]]** | An alternative approach for building HTTP APIs that adds more structure and features than minimal APIs |
| **gRPC APIs** | Efficient binary APIs for server-to-server communication using the gRPC protocol |
| **[[Razor Pages]]** | Page-based server-rendered applications |
| **[[Controllers Overview\|MVC Controllers]]** | Similar to Razor Pages -- used for server-based applications but without the page-based paradigm |
| **Blazor WebAssembly** | A browser-based SPA framework using the WebAssembly standard, similar to JavaScript frameworks such as Angular, React, and Vue |
| **Blazor Server** | Stateful applications rendered on the server that send UI events and page updates over WebSockets to provide the feel of a client-side SPA but with the ease of development of a server-rendered application |

### Communication Patterns

These application models fall into three distinct ==communication patterns== based on what the server sends back:

| Pattern | Request Type | Response Format | Example |
|---|---|---|---|
| **Traditional Web App** | ==Synchronous== HTTP request | Full ==HTML== web page | MVC, Razor Pages |
| **REST API** | ==Asynchronous== HTTP request | Partial data as ==JSON or XML== | Web APIs, Minimal APIs |
| **RPC Service** | Synchronous or asynchronous via HTTP | Data as ==JSON, XML, or binary== | gRPC |

```
┌─── Traditional Web App ──────────────────────────────────────────────┐
│                                                                      │
│  📱🖥️ Client         Sync HTTP request          🖥️ Server           │
│              ─────────────────────────────►    (MVC / Razor Pages)   │
│              ◄─────────────────────────────                          │
│                  Response: Full HTML page                             │
│                  ┌──────────────────┐                                 │
│                  │  <html>          │                                 │
│                  │    <body>...</body│                                │
│                  │  </html>         │                                 │
│                  └──────────────────┘                                 │
└──────────────────────────────────────────────────────────────────────┘

┌─── REST API ─────────────────────────────────────────────────────────┐
│                                                                      │
│  📱🖥️ Client        Async HTTP request          🖥️ Server           │
│              ─────────────────────────────►    (Web API / Minimal)   │
│              ◄─────────────────────────────                          │
│                  Response: JSON or XML                                │
│                  ┌──────────────────┐                                 │
│                  │ { "id": 1,       │                                 │
│                  │   "name": "..." }│                                 │
│                  └──────────────────┘                                 │
└──────────────────────────────────────────────────────────────────────┘

┌─── RPC Service (gRPC) ───────────────────────────────────────────────┐
│                                                                      │
│  🖥️ Service A     Sync/Async via HTTP/2         🖥️ Service B        │
│              ─────────────────────────────►    (gRPC server)         │
│              ◄─────────────────────────────                          │
│                  Response: Binary (Protobuf)                          │
│                  ┌──────────────────┐                                 │
│                  │ 0x0A 0x07 0x4F...│                                 │
│                  │ (binary data)    │                                 │
│                  └──────────────────┘                                 │
└──────────────────────────────────────────────────────────────────────┘
```

**Traditional Web App** — the browser sends a request, the server builds an entire HTML page and sends it back. Every navigation is a full page reload. This is ==server-side rendering (SSR)==. See [[#Server-Side Rendering vs Client-Side Rendering]].

**REST API** — the browser (or mobile app) sends a request and gets back ==just the data== (usually JSON). A JavaScript framework (React, Angular, Vue) or mobile app then renders the UI on the client side. The server doesn't know or care how the data is displayed.

**RPC Service (gRPC)** — used for ==server-to-server== communication, not browser-to-server. A backend service calls another backend service directly using efficient binary serialization (Protocol Buffers). Clients are typically other microservices, not browsers.

> [!ad-tip] Which Pattern to Choose?
> - Building a content website or form-heavy app? → **Traditional web app** (Razor Pages / MVC)
> - Building a SPA or mobile backend? → **REST API** (Minimal APIs / Web API Controllers)
> - Microservices calling each other? → **gRPC**
> - Need all of the above? → ASP.NET Core lets you mix them in the same project

### MVC Web Applications

The traditional Model-View-Controller pattern for server-rendered HTML applications with Razor views.

```csharp
// Controllers/OrderController.cs
public class OrderController : Controller
{
    private readonly IOrderService _orderService;

    public OrderController(IOrderService orderService)
    {
        _orderService = orderService;
    }

    public async Task<IActionResult> Index()
    {
        var orders = await _orderService.GetRecentOrdersAsync();
        return View(orders);
    }

    [HttpPost]
    public async Task<IActionResult> Create(CreateOrderViewModel model)
    {
        if (!ModelState.IsValid)
            return View(model);

        await _orderService.CreateOrderAsync(model);
        return RedirectToAction(nameof(Index));
    }
}
```

### Web APIs (REST)

Build HTTP APIs that return JSON, used by SPAs, mobile apps, and other services.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly InventoryContext _context;

    public ProductsController(InventoryContext context)
    {
        _context = context;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<Product>>> GetAll()
    {
        return await _context.Products
            .Where(p => p.IsActive)
            .OrderBy(p => p.Name)
            .ToListAsync();
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetById(int id)
    {
        var product = await _context.Products.FindAsync(id);
        return product is null ? NotFound() : Ok(product);
    }
}
```

### Razor Pages

A page-focused model that simplifies server-rendered scenarios. Each page is self-contained with its own model.

```csharp
// Pages/Orders/Index.cshtml.cs
public class IndexModel : PageModel
{
    private readonly IOrderService _orderService;

    public IndexModel(IOrderService orderService)
    {
        _orderService = orderService;
    }

    public IList<Order> Orders { get; set; } = default!;

    public async Task OnGetAsync()
    {
        Orders = await _orderService.GetRecentOrdersAsync();
    }
}
```

### Blazor

Build interactive web UIs using C# instead of JavaScript. Blazor comes in two hosting models:

- **Blazor Server** -- UI logic runs on the server, updates sent over SignalR
- **Blazor WebAssembly** -- C# runs directly in the browser via WebAssembly
- **Blazor United / Auto** (.NET 8+) -- combines server and WebAssembly rendering

> [!example] Blazor Component
> ```csharp
> @page "/orders"
> @inject IOrderService OrderService
> 
> <h3>Recent Orders</h3>
> 
> @if (_orders is null)
> {
>     <p>Loading...</p>
> }
> else
> {
>     <table class="table">
>         @foreach (var order in _orders)
>         {
>             <tr>
>                 <td>@order.OrderNumber</td>
>                 <td>@order.CustomerName</td>
>                 <td>@order.Total.ToString("C")</td>
>             </tr>
>         }
>     </table>
> }
> 
> @code {
>     private List<Order>? _orders;
> 
>     protected override async Task OnInitializedAsync()
>     {
>         _orders = await OrderService.GetRecentOrdersAsync();
>     }
> }
> ```

### gRPC Services

High-performance RPC framework using Protocol Buffers for service-to-service communication.

```csharp
// Services/OrderGrpcService.cs
public class OrderGrpcService : OrderProto.OrderProtoBase
{
    private readonly IOrderRepository _repository;

    public OrderGrpcService(IOrderRepository repository)
    {
        _repository = repository;
    }

    public override async Task<OrderResponse> GetOrder(
        OrderRequest request, ServerCallContext context)
    {
        var order = await _repository.GetByIdAsync(request.OrderId);
        return new OrderResponse
        {
            OrderId = order.Id,
            CustomerName = order.CustomerName,
            Total = (double)order.Total
        };
    }
}
```

### SignalR (Real-Time)

Real-time web functionality for push notifications, chat, live dashboards.

```csharp
// Hubs/OrderNotificationHub.cs
public class OrderNotificationHub : Hub
{
    public async Task NotifyOrderStatusChanged(int orderId, string newStatus)
    {
        await Clients.All.SendAsync("OrderStatusUpdated", orderId, newStatus);
    }
}

// In Program.cs
builder.Services.AddSignalR();
app.MapHub<OrderNotificationHub>("/hubs/orders");
```

> [!summary] Section Summary
> - **MVC**: server-rendered HTML apps with Controllers and Razor views
> - **Web APIs**: JSON-returning endpoints for SPAs, mobile, and microservices
> - **Razor Pages**: page-centric model for simpler server-rendered scenarios
> - **Blazor**: interactive UIs in C# via Server, WebAssembly, or Auto rendering
> - **gRPC**: high-performance RPC with Protocol Buffers for service-to-service calls
> - **SignalR**: real-time bidirectional communication (chat, notifications, dashboards)
