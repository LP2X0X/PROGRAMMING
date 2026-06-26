---
tags: [csharp, asp-net-core, fundamentals, web]
---

# ASP.NET Core Overview

> [!ad-note] About This Note
> This note covers the foundations of ASP.NET Core -- what it is, why it was built from scratch, what you can build with it, and where it sits in the broader .NET ecosystem. This is the starting point before diving into [[Project Structure]], [[Hosting Model]], [[Program.cs and Startup]], and [[Environments]].

---

## Table of Contents

- [[#What Is ASP.NET Core]]
- [[#History -- From ASP.NET Framework to ASP.NET Core]]
- [[#Key Characteristics]]
- [[#What You Can Build]]
- [[#Communication Patterns]]
- [[#Server-Side Rendering vs Client-Side Rendering]]
- [[#ASP.NET Core vs ASP.NET Framework]]
- [[#The .NET Ecosystem and Release Cadence]]
- [[#The Microsoft.AspNetCore.App Metapackage]]
- [[#Where ASP.NET Core Fits in the .NET Stack]]
- [[#Your First ASP.NET Core Application]]
- [[#The Request Pipeline at a Glance]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## What Is ASP.NET Core

ASP.NET Core is a **cross-platform, high-performance, open-source framework** for building modern, cloud-enabled web applications and services. It runs on .NET (formerly .NET Core) and is designed from the ground up to be modular, testable, and lightweight.

Unlike its predecessor (ASP.NET on .NET Framework), ASP.NET Core is not tied to Windows or IIS. You can run it on Windows, Linux, and macOS, deploy it to Docker containers, and host it behind any reverse proxy -- not just IIS.

```csharp
// The simplest possible ASP.NET Core application
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello from ASP.NET Core!");

app.Run();
```

> [!tip] Minimal APIs
> Starting with .NET 6, ASP.NET Core supports **Minimal APIs** -- a streamlined way to build HTTP endpoints with minimal ceremony. The example above is a complete, runnable web application in just four lines.

### Core Design Principles

ASP.NET Core was designed around several foundational principles:

1. **Pay-for-what-you-use**: Only the middleware and services you explicitly add are included in the pipeline. No bloated default stack.
2. **Dependency injection first**: DI is built into the framework at every level, not bolted on as an afterthought.
3. **Configuration flexibility**: Multiple configuration sources (JSON, environment variables, command line, Azure Key Vault, user secrets) are unified under a single API.
4. **Testability**: Every component can be mocked and tested in isolation thanks to interface-driven design.
5. **Performance**: ASP.NET Core consistently ranks among the fastest web frameworks in the TechEmpower benchmarks.

> [!summary] Section Summary
> - ASP.NET Core is a cross-platform, high-performance web framework running on .NET
> - It is modular, testable, and built with dependency injection at its core
> - It supports Minimal APIs (from .NET 6+) for lightweight endpoint definitions
> - It follows a pay-for-what-you-use model -- no unnecessary defaults in the pipeline

---

## History -- From ASP.NET Framework to ASP.NET Core

> [!warning] Common Misconception
> ASP.NET Core is **NOT** an upgrade or evolution of ASP.NET Framework. It is a **complete ground-up rewrite**. Code from ASP.NET Framework (WebForms, classic MVC with `System.Web`) does not directly port to ASP.NET Core.

### The Legacy: ASP.NET on .NET Framework

ASP.NET was introduced in 2002 as part of the .NET Framework. Over the years it grew to include:

- **Web Forms** (2002) -- event-driven, drag-and-drop web development
- **ASP.NET MVC** (2009) -- Model-View-Controller pattern, more control over HTML
- **ASP.NET Web API** (2012) -- HTTP services for REST APIs
- **SignalR** (2013) -- real-time web communication

All of these were tightly coupled to `System.Web.dll` and IIS on Windows. This coupling meant:

- Windows-only deployment
- Heavy runtime overhead from `System.Web`
- Monolithic framework -- you got everything whether you needed it or not
- Slow iteration cycles tied to .NET Framework releases

### The Rewrite: ASP.NET Core

In 2016, Microsoft released **ASP.NET Core 1.0** alongside **.NET Core 1.0**. The goals were clear:

- Break free from `System.Web` and IIS dependency
- Run on Linux and macOS
- Achieve dramatically better performance
- Enable side-by-side versioning (multiple .NET versions on the same machine)
- Ship as NuGet packages with independent release cycles

### Timeline of Major Releases

| Version | Year | Key Highlights |
|---|---|---|
| ASP.NET Core 1.0 | 2016 | Initial cross-platform release |
| ASP.NET Core 2.0 | 2017 | Razor Pages, metapackage introduced |
| ASP.NET Core 3.0 | 2019 | gRPC support, Blazor Server, dropped .NET Framework target |
| .NET 5 (unified) | 2020 | Merged .NET Core and .NET Framework naming |
| .NET 6 | 2021 | Minimal APIs, Hot Reload (LTS) |
| .NET 7 | 2022 | Rate limiting, output caching, HTTP/3 |
| .NET 8 | 2023 | Native AOT for APIs, Blazor United (LTS) |
| .NET 9 | 2024 | HybridCache, OpenAPI built-in, perf improvements |

> [!ad-note] Naming Clarification
> After .NET Core 3.1, Microsoft dropped "Core" from the product name. **.NET 5** and later are simply called ".NET" -- but the web framework is still commonly referred to as "ASP.NET Core" to distinguish it from the legacy ASP.NET Framework.

> [!summary] Section Summary
> - ASP.NET Core is a ground-up rewrite, not an upgrade of ASP.NET Framework
> - The legacy framework was tightly coupled to `System.Web.dll` and Windows/IIS
> - ASP.NET Core 1.0 launched in 2016 to break those dependencies
> - After .NET Core 3.1, the naming unified to just ".NET" (5, 6, 7, 8, 9...)
> - Code from ASP.NET Framework (especially WebForms) does not directly port over

---

## Key Characteristics

### Cross-Platform

ASP.NET Core applications run anywhere .NET runs:

- **Windows** -- IIS, Windows Services, Kestrel standalone
- **Linux** -- systemd services, Docker containers, Kubernetes
- **macOS** -- development and production

```bash
# Create and run an ASP.NET Core app on any OS
dotnet new webapi -n OrderService
cd OrderService
dotnet run
```

### Open Source

The entire ASP.NET Core framework is open source under the MIT license. The source code is on GitHub at `dotnet/aspnetcore`. You can read the source, file issues, and submit pull requests.

### Modular Architecture

ASP.NET Core uses a **middleware pipeline** where you compose your application from small, focused components. Nothing is included by default -- you opt in to what you need.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Only add the services you need
builder.Services.AddControllers();
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
builder.Services.AddDbContext<InventoryContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Inventory")));

var app = builder.Build();

// Only add the middleware you need
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### High Performance

ASP.NET Core is built on **Kestrel**, a high-performance, cross-platform HTTP server. Key performance features include:

- Kestrel uses asynchronous I/O throughout
- Memory-efficient request processing with `System.IO.Pipelines`
- `Span<T>` and `Memory<T>` used extensively to reduce allocations
- Support for HTTP/2 and HTTP/3
- Native AOT compilation support (from .NET 8) for minimal startup time

> [!tip] TechEmpower Benchmarks
> ASP.NET Core consistently places in the top tier of the TechEmpower web framework benchmarks, often outperforming frameworks from other ecosystems like Node.js Express and Spring Boot in plaintext and JSON serialization tests.

### Built-in Dependency Injection

Unlike ASP.NET Framework where DI was optional and required third-party containers (Unity, Autofac, Ninject), ASP.NET Core has a **first-class DI container** built in. Every service in the framework is registered and resolved through DI.

```csharp
// Registering services
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();
```

See [[17.03 - Dependency Injection|Dependency Injection]] for a deep dive into service lifetimes and registration patterns.

### Unified Configuration System

ASP.NET Core replaces the old `web.config` / `ConfigurationManager` approach with a layered configuration system:

```json
// appsettings.json
{
  "ConnectionStrings": {
    "InventoryDb": "Server=localhost;Database=Inventory;Trusted_Connection=true;"
  },
  "OrderProcessing": {
    "MaxRetryAttempts": 3,
    "TimeoutSeconds": 30
  }
}
```

```csharp
// Reading configuration with the Options pattern
public class OrderProcessingOptions
{
    public int MaxRetryAttempts { get; set; }
    public int TimeoutSeconds { get; set; }
}

// In Program.cs
builder.Services.Configure<OrderProcessingOptions>(
    builder.Configuration.GetSection("OrderProcessing"));

// In a service -- injected via DI
public class OrderService
{
    private readonly OrderProcessingOptions _options;

    public OrderService(IOptions<OrderProcessingOptions> options)
    {
        _options = options.Value;
    }
}
```

> [!summary] Section Summary
> - Cross-platform: runs on Windows, Linux, and macOS
> - Open source under MIT license, hosted on GitHub
> - Modular middleware pipeline -- you compose only what you need
> - High performance via Kestrel, async I/O, Span/Memory, and HTTP/2-3 support
> - Built-in DI container replaces the need for third-party IoC containers
> - Unified configuration system with JSON files, environment variables, and the Options pattern

---

## What You Can Build

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

---

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

---

## Server-Side Rendering vs Client-Side Rendering

The application models above fall into two rendering strategies. Understanding this distinction is fundamental to choosing the right approach.

### Server-Side Rendering (SSR)

With **Razor Pages** and **MVC Controllers**, your C# code on the server builds the complete HTML before sending it to the browser. The browser receives a finished document and simply displays it.

```
Browser: "GET /products/5"
    │
    ▼
Server:  1. Fetches product from database
         2. Runs Razor template with the data
         3. Produces complete HTML string
         4. Sends finished HTML back
    │
    ▼
Browser: Receives HTML, displays it immediately
```

### Client-Side Rendering (CSR)

With **Web API Controllers** and **Minimal APIs**, the server only returns JSON data. A JavaScript framework (React, Vue, Angular) running in the browser receives that data and builds the HTML itself.

```
Browser: Loads a JavaScript app (React, Vue, etc.)
React:   "GET /api/products/5"
    │
    ▼
Server:  1. Fetches product from database
         2. Returns JSON: {"name":"Mouse","price":29.99}
    │
    ▼
React:   Receives JSON, builds the HTML in the browser
```

### Comparison

| | SSR (Razor Pages / MVC) | CSR (API + JS Framework) |
|---|---|---|
| **Who builds HTML?** | Server (C#) | Browser (JavaScript) |
| **First page load** | Fast -- HTML arrives ready to display | Slower -- must download the JS app first |
| **Interactivity** | Each action = full page reload | Smooth, app-like experience |
| **SEO** | Great -- search engines see complete HTML | Harder -- requires extra setup |
| **Complexity** | One codebase (C#) | Two codebases (C# API + JS frontend) |
| **Feels like** | Traditional website | Desktop/mobile app in the browser |
| **Best for** | Content sites, forms, admin panels | Dashboards, highly interactive UIs |

> [!ad-note] Blazor Blurs the Line
> Blazor does not fit neatly into either category. **Blazor Server** renders on the server but pushes incremental UI updates over a WebSocket (SignalR) -- no full page reloads. **Blazor WebAssembly** runs C# directly in the browser, making it CSR but without JavaScript. **Blazor Auto** (.NET 8+) starts with server rendering for fast first load, then switches to WebAssembly for interactivity.

> [!ad-note] Desktop Developers -- Mental Model
> If you come from WinForms or WPF, CSR will feel more familiar -- the JavaScript framework works like a UI toolkit (components, state, event handlers). SSR is more like generating a printable document on every user action. In neither case does your server-side C# directly render pixels -- the **browser** is always the GUI runtime.

> [!summary] Section Summary
> - **SSR** (Razor Pages, MVC): server builds complete HTML, browser just displays it. Simpler, one C# codebase, great SEO.
> - **CSR** (Web APIs + React/Vue/Angular): server sends JSON, browser builds the UI. More interactive, but two codebases.
> - **Blazor** bridges both -- C# in the browser (WebAssembly) or server-pushed updates (Server/Auto).

---

## ASP.NET Core vs ASP.NET Framework

> [!warning] Migration Decision
> If you are maintaining an existing ASP.NET Framework app, you do not have to migrate. But all **new** projects should use ASP.NET Core. ASP.NET Framework is in maintenance mode -- it receives security patches only.

| Feature | ASP.NET Framework | ASP.NET Core |
|---|---|---|
| **Platform** | Windows only | Windows, Linux, macOS |
| **Web Server** | IIS only | Kestrel (cross-platform) + IIS/Nginx/Apache as reverse proxy |
| **Dependency Injection** | Not built-in (third-party required) | Built-in DI container |
| **Configuration** | `web.config` / `ConfigurationManager` | JSON, env vars, command line, user secrets, Azure Key Vault |
| **Performance** | Moderate | Top-tier (TechEmpower benchmarks) |
| **Pipeline** | `HttpModule` / `HttpHandler` (monolithic) | Middleware pipeline (composable) |
| **Hosting** | IIS process (`w3wp.exe`) | Self-hosted, IIS, Docker, systemd |
| **Side-by-Side** | Machine-wide .NET Framework | App-local .NET runtime |
| **Open Source** | Partially | Fully open source (MIT) |
| **Development Status** | Maintenance mode (security patches only) | Active development |
| **Deployment** | Windows Server + IIS | Anywhere: cloud, containers, edge |
| **Minimal APIs** | Not available | Supported (.NET 6+) |
| **Native AOT** | Not available | Supported (.NET 8+) |

> [!ad-note] What Cannot Be Migrated Easily
> Some ASP.NET Framework technologies have **no direct equivalent** in ASP.NET Core:
> - **Web Forms** -- no equivalent; use Blazor or Razor Pages instead
> - **WCF Server** -- use gRPC or REST APIs instead (CoreWCF exists as a community port)
> - **Windows-specific APIs** -- `System.Drawing`, `System.DirectoryServices`, etc. require Windows Compatibility Pack or alternatives

> [!summary] Section Summary
> - ASP.NET Core wins on platform support, performance, DI, and modern tooling
> - ASP.NET Framework is Windows/IIS-only and in maintenance mode
> - New projects should always target ASP.NET Core
> - Some legacy technologies (Web Forms, WCF) have no direct ASP.NET Core equivalent
> - Migration is possible but requires rewriting, not just upgrading

---

## The .NET Ecosystem and Release Cadence

### Release Schedule

Microsoft follows a predictable annual release cadence:

- **Even-numbered releases** (.NET 6, 8, 10) are **LTS (Long-Term Support)** -- 3 years of support
- **Odd-numbered releases** (.NET 7, 9) are **STS (Standard-Term Support)** -- 18 months of support

```
.NET 6 (LTS) -- Nov 2021 to Nov 2024
.NET 7 (STS) -- Nov 2022 to May 2024
.NET 8 (LTS) -- Nov 2023 to Nov 2026
.NET 9 (STS) -- Nov 2024 to May 2026
.NET 10 (LTS) -- Nov 2025 to Nov 2028  (projected)
```

> [!tip] Choosing a Version
> For production applications, prefer the latest **LTS** release unless you specifically need a feature from the latest STS release. As of mid-2026, **.NET 8** is the current LTS and **.NET 10** is newly released as the next LTS.

### SDK and Runtime

The .NET ecosystem has two main components:

- **.NET SDK** -- includes the compiler, CLI tools (`dotnet` command), and project templates. Used for development.
- **.NET Runtime** -- the minimal runtime needed to execute a .NET application. Used for deployment.

```bash
# Check installed SDKs and runtimes
dotnet --list-sdks
dotnet --list-runtimes

# Check current SDK version
dotnet --version
```

### The dotnet CLI

The `dotnet` CLI is the primary tool for creating, building, running, and publishing ASP.NET Core apps:

```bash
# Create a new web API project
dotnet new webapi -n InventoryApi

# Restore NuGet packages
dotnet restore

# Build the project
dotnet build

# Run with hot reload
dotnet watch run

# Publish for deployment
dotnet publish -c Release -o ./publish
```

> [!summary] Section Summary
> - .NET follows an annual release cycle: even = LTS (3 years), odd = STS (18 months)
> - Prefer the latest LTS release for production workloads
> - The SDK is for development; the Runtime is the minimal deployment target
> - The `dotnet` CLI handles project creation, building, running, and publishing

---

## The Microsoft.AspNetCore.App Metapackage

ASP.NET Core ships as a **shared framework** via the `Microsoft.AspNetCore.App` metapackage. This package bundles all the core ASP.NET Core libraries:

- MVC, Razor Pages, Razor runtime compilation
- Kestrel web server
- Authentication and authorization
- SignalR
- gRPC
- Logging, configuration, DI
- Health checks, CORS, response caching

### How It Works

You do **not** need to add an explicit NuGet reference to `Microsoft.AspNetCore.App`. When your project targets `Microsoft.NET.Sdk.Web`, the framework reference is included implicitly:

```xml
<!-- Your .csproj file -->
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

> [!ad-note] Sdk="Microsoft.NET.Sdk.Web"
> The `Microsoft.NET.Sdk.Web` SDK type is what triggers the implicit framework reference. If you create a class library that needs ASP.NET Core types (e.g., for shared controllers), you must add an explicit framework reference:
> ```xml
> <ItemGroup>
>   <FrameworkReference Include="Microsoft.AspNetCore.App" />
> </ItemGroup>
> ```

### What Is NOT in the Shared Framework

Some packages are **not** included in the shared framework and must be added as separate NuGet packages:

| Package | Purpose |
|---|---|
| `Microsoft.EntityFrameworkCore.*` | Database access (EF Core) |
| `Microsoft.AspNetCore.Authentication.JwtBearer` | JWT token authentication |
| `Microsoft.AspNetCore.Identity.EntityFrameworkCore` | ASP.NET Core Identity with EF |
| `Swashbuckle.AspNetCore` | Swagger/OpenAPI generation |
| `Microsoft.AspNetCore.OpenApi` | Built-in OpenAPI (.NET 9+) |
| `Serilog.AspNetCore` | Structured logging with Serilog |

```xml
<!-- Adding packages not in the shared framework -->
<ItemGroup>
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
  <PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
</ItemGroup>
```

> [!summary] Section Summary
> - `Microsoft.AspNetCore.App` is the shared framework containing all core ASP.NET Core libraries
> - It is included implicitly when using `Microsoft.NET.Sdk.Web` -- no explicit NuGet reference needed
> - Class libraries that need ASP.NET Core types must add an explicit `<FrameworkReference>`
> - EF Core, JWT auth, Identity, and third-party logging are NOT included and require separate NuGet packages

---

## Where ASP.NET Core Fits in the .NET Stack

ASP.NET Core does not exist in isolation. It sits within a broader ecosystem of libraries and frameworks:

```
+-----------------------------------------------------+
|                Your ASP.NET Core App                 |
|  (Controllers, Minimal APIs, Blazor, Razor Pages)    |
+-----------------------------------------------------+
|                ASP.NET Core Framework                |
|  (Routing, Middleware, Auth, Model Binding, Filters)  |
+-----------------------------------------------------+
|              .NET Base Class Libraries                |
|  (Collections, IO, Networking, JSON, Cryptography)    |
+-----------------------------------------------------+
|                  .NET Runtime (CLR)                   |
|  (GC, JIT, Type System, Threading)                    |
+-----------------------------------------------------+
|                Operating System                       |
|  (Windows, Linux, macOS)                              |
+-----------------------------------------------------+
```

### Key Companion Libraries

| Library | Relationship |
|---|---|
| [[DbContext\|Entity Framework Core]] | ORM for database access; integrates via DI and `DbContext` |
| [[17.03 - Dependency Injection\|DI Container]] | Built into ASP.NET Core; resolves all services |
| [[Hosting Model\|Kestrel / Hosting]] | The HTTP server that powers ASP.NET Core |
| Serilog / NLog | Structured logging providers that plug into the logging abstraction |
| MediatR | Mediator pattern library commonly used with CQRS in ASP.NET Core |
| FluentValidation | Replaces or supplements DataAnnotations for request validation |
| AutoMapper / Mapster | Object-to-object mapping between DTOs and domain models |

> [!example] Typical Service Registration
> A real-world ASP.NET Core `Program.cs` brings together multiple libraries:
> ```csharp
> var builder = WebApplication.CreateBuilder(args);
> 
> // ASP.NET Core
> builder.Services.AddControllers();
> builder.Services.AddEndpointsApiExplorer();
> 
> // Entity Framework Core
> builder.Services.AddDbContext<InventoryContext>(options =>
>     options.UseSqlServer(builder.Configuration.GetConnectionString("InventoryDb")));
> 
> // Application services
> builder.Services.AddScoped<IOrderService, OrderService>();
> builder.Services.AddScoped<IInventoryService, InventoryService>();
> 
> // Third-party: Serilog
> builder.Host.UseSerilog((context, config) =>
>     config.ReadFrom.Configuration(context.Configuration));
> 
> var app = builder.Build();
> 
> app.UseSerilogRequestLogging();
> app.UseHttpsRedirection();
> app.UseAuthorization();
> app.MapControllers();
> 
> app.Run();
> ```

> [!summary] Section Summary
> - ASP.NET Core sits on top of the .NET BCL and CLR runtime
> - It integrates tightly with EF Core for data access and the built-in DI container
> - Third-party libraries (Serilog, MediatR, FluentValidation) plug in through standardized abstractions
> - A typical `Program.cs` composes ASP.NET Core, EF Core, and application services together

---

## Your First ASP.NET Core Application

### Creating the Project

```bash
# Create a new Web API project
dotnet new webapi -n OrderApi --use-controllers

# Navigate to the project
cd OrderApi
```

This generates the following structure (see [[Project Structure]] for a detailed breakdown):

```
OrderApi/
  Properties/
    launchSettings.json      -- Development launch profiles
  Controllers/
    WeatherForecastController.cs
  appsettings.json           -- Default configuration
  appsettings.Development.json -- Environment-specific overrides
  OrderApi.csproj            -- Project file
  Program.cs                 -- Application entry point
```

### Understanding Program.cs

The `Program.cs` file is the entry point. See [[Program.cs and Startup]] for a deep dive.

```csharp
var builder = WebApplication.CreateBuilder(args);

// === Service Registration Phase ===
// Add services to the DI container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// === Middleware Pipeline Phase ===
// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

> [!ad-note] Two Phases
> Every ASP.NET Core app has two distinct phases:
> 1. **Service registration** (`builder.Services.Add...`) -- configure DI container
> 2. **Middleware pipeline** (`app.Use...`, `app.Map...`) -- configure HTTP request processing
>
> This separation is important. Services are registered before `builder.Build()` is called. Middleware is configured after.

### Running the Application

```bash
# Run with default settings
dotnet run

# Run with hot reload for development
dotnet watch run

# Run in a specific environment
dotnet run --environment Development
```

The app starts on `https://localhost:5001` and `http://localhost:5000` by default. See [[Environments]] for how environment-specific configuration works.

> [!summary] Section Summary
> - Use `dotnet new webapi` to scaffold a new API project
> - `Program.cs` has two phases: service registration and middleware pipeline configuration
> - Services are added before `builder.Build()`; middleware is configured after
> - `dotnet watch run` enables hot reload during development
> - Default URLs are `https://localhost:5001` and `http://localhost:5000`

---

## The Request Pipeline at a Glance

ASP.NET Core processes every HTTP request through a **middleware pipeline**. Each middleware component can:

1. Handle the request and short-circuit the pipeline
2. Pass the request to the next middleware in the chain
3. Execute logic before AND after the next middleware (wrapping pattern)

```
Request --> Middleware 1 --> Middleware 2 --> Middleware 3 --> Endpoint
              |                |                |
              v                v                v
Response <-- Middleware 1 <-- Middleware 2 <-- Middleware 3
```

> [!warning] Order Matters
> Middleware order is critical. For example, `UseAuthentication()` must come before `UseAuthorization()`, and both must come before `MapControllers()`. Incorrect ordering leads to subtle bugs -- such as authorization running before the user's identity is established.

### Common Middleware Order

```csharp
var app = builder.Build();

// 1. Exception handling (outermost -- catches everything)
app.UseExceptionHandler("/error");

// 2. HSTS (HTTP Strict Transport Security)
app.UseHsts();

// 3. HTTPS redirection
app.UseHttpsRedirection();

// 4. Static files (short-circuits for CSS, JS, images)
app.UseStaticFiles();

// 5. Routing (matches the request to an endpoint)
app.UseRouting();

// 6. CORS (must be between routing and auth)
app.UseCors();

// 7. Authentication (who are you?)
app.UseAuthentication();

// 8. Authorization (are you allowed?)
app.UseAuthorization();

// 9. Endpoint execution
app.MapControllers();
app.MapRazorPages();
```

> [!tip] Custom Middleware
> You can write your own middleware for cross-cutting concerns like request logging, correlation IDs, or tenant resolution:
> ```csharp
> app.Use(async (context, next) =>
> {
>     var correlationId = Guid.NewGuid().ToString();
>     context.Response.Headers["X-Correlation-Id"] = correlationId;
>
>     // Before the next middleware
>     var stopwatch = Stopwatch.StartNew();
>
>     await next(context);
>
>     // After the next middleware
>     stopwatch.Stop();
>     Console.WriteLine(
>         $"Request {context.Request.Path} took {stopwatch.ElapsedMilliseconds}ms");
> });
> ```

> [!summary] Section Summary
> - Every request flows through a composable middleware pipeline
> - Each middleware can handle, pass through, or wrap the request
> - Middleware order is critical -- authentication before authorization before endpoints
> - Custom middleware is straightforward to write for cross-cutting concerns
> - Static files middleware short-circuits the pipeline for non-dynamic content

---

## Comprehensive Summary

> [!tip] Complete Summary
> **ASP.NET Core** is a cross-platform, high-performance, open-source web framework that is a **complete ground-up rewrite** of the legacy ASP.NET Framework. It runs on .NET 6/8/9+ and supports Windows, Linux, and macOS.
>
> **What you can build**: MVC web apps, REST APIs, Razor Pages, Blazor interactive UIs, gRPC services, and SignalR real-time apps -- all under one unified framework.
>
> **Key design principles**: modular middleware pipeline (pay-for-what-you-use), built-in dependency injection, unified configuration system (JSON + env vars + Options pattern), and testability through interface-driven design.
>
> **Performance**: ASP.NET Core ranks among the fastest web frameworks globally, powered by the Kestrel HTTP server, async I/O, `Span<T>`/`Memory<T>` optimizations, and support for HTTP/2, HTTP/3, and Native AOT.
>
> **Ecosystem**: The `Microsoft.AspNetCore.App` shared framework is included implicitly via the `Microsoft.NET.Sdk.Web` SDK. Additional libraries like EF Core, JWT authentication, and structured logging are added as separate NuGet packages.
>
> **Release cadence**: Even-numbered releases (.NET 8, 10) are LTS with 3-year support; odd-numbered releases (.NET 9) are STS with 18-month support. Always prefer the latest LTS for production.
>
> **Migration note**: ASP.NET Framework is in maintenance mode. All new projects should use ASP.NET Core. Legacy technologies like Web Forms and WCF Server have no direct equivalent -- use Blazor/Razor Pages and gRPC respectively.
>
> **Next steps**: Explore [[Project Structure]] for how ASP.NET Core apps are organized, [[Hosting Model]] for Kestrel and reverse proxy configuration, [[Program.cs and Startup]] for the application bootstrap process, and [[Environments]] for managing Development/Staging/Production settings.

---

## Related Topics

- [[Project Structure]] -- how ASP.NET Core projects are organized on disk
- [[Hosting Model]] -- Kestrel, IIS integration, and reverse proxy patterns
- [[Program.cs and Startup]] -- the application entry point and bootstrap process
- [[Environments]] -- Development, Staging, Production configuration
- [[17.03 - Dependency Injection|Dependency Injection]] -- the built-in DI container and service lifetimes
- [[DbContext|Entity Framework Core]] -- ORM integration with ASP.NET Core
- [[Middleware]] -- deep dive into the request pipeline
- [[Configuration and Options Pattern]] -- layered configuration and strongly-typed options
- [[Authentication and Authorization]] -- identity, JWT, cookies, policies
- [[Minimal APIs]] -- lightweight endpoint definitions without controllers
