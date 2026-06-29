---
tags: [csharp, asp-net-core, hosting, kestrel]
---


Docker is the most common deployment target for ASP.NET Core applications in modern cloud environments. Microsoft provides official base images optimized for building and running .NET apps.

### Basic Dockerfile

```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

COPY ["OrderService.csproj", "./"]
RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o /app/publish

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .

EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
ENV ASPNETCORE_ENVIRONMENT=Production

ENTRYPOINT ["dotnet", "OrderService.dll"]
```

### Key Docker Images

| Image | Purpose | Size |
|---|---|---|
| `mcr.microsoft.com/dotnet/sdk:8.0` | Building and publishing | ~800 MB |
| `mcr.microsoft.com/dotnet/aspnet:8.0` | Running web apps | ~220 MB |
| `mcr.microsoft.com/dotnet/runtime:8.0` | Running console/worker apps | ~190 MB |
| `mcr.microsoft.com/dotnet/aspnet:8.0-alpine` | Minimal runtime (Alpine Linux) | ~110 MB |

> [!tip] Use Multi-Stage Builds
> Always use multi-stage builds. The SDK image is large (800 MB+) and contains compilers, NuGet tools, and other build-time dependencies. The final runtime image should use `aspnet` or `runtime` to keep the container small and reduce the attack surface.

### Docker Compose for Development

```yaml
version: '3.8'
services:
  orderservice:
    build: .
    ports:
      - "5000:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__InventoryDb=Server=db;Database=Inventory;User=sa;Password=Dev@Pass123
    depends_on:
      - db

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=Dev@Pass123
    ports:
      - "1433:1433"
```

> [!ad-note] .NET 8 Port Change
> Starting with .NET 8, the default port in official Docker images changed from `80` to `8080`. If you are upgrading from .NET 6 or 7, update your `EXPOSE` directive and `ASPNETCORE_URLS` accordingly. Existing health checks, load balancer configurations, and Kubernetes readiness probes may also need updating.

> [!summary] Section Summary
> - Microsoft provides official Docker images for building (`sdk`) and running (`aspnet`) .NET applications
> - Multi-stage builds keep final images small by excluding build-time dependencies
> - .NET 8 images default to port 8080 instead of port 80
> - Docker Compose simplifies local development with database and service dependencies
