---
tags:
  - NIS
  - DCS
  - LCS
  - architecture
  - reference
---

# NIS — Generic Technical Knowledge (DCS / LCS)

Pure engineering concepts, patterns, frameworks, and technologies used in the NIS codebase. No business-domain terms.

---

## Architecture & Design Patterns

| Keyword | Detail |
|---|---|
| **EventBus (Pub/Sub)** | In-process topic-based publish/subscribe. Components implement `ISubscriber<T>` / `IPublisher<T>`. Topics are string-keyed, dispatched via `TopicContainer` backed by `ConcurrentDictionary`. Each process (DCS, LCS) owns its own EventBus instance |
| **EventNetwork (TCP messaging)** | Cross-process TCP socket layer built on `NetworkClient` / `NetworkServer` / `NetworkSession`. Messages serialized with MessagePack, framed and routed by type. Server tracks sessions in `ConcurrentDictionary` |
| **Driver pattern** | Each external endpoint (hardware, service, protocol) is wrapped in a dedicated Driver class that encapsulates connection lifecycle, message framing, reconnect logic, and adapter abstraction (real vs virtual) |
| **Adapter pattern** | Hardware drivers use `RealAdapter` / `VirtualAdapter` pairs behind a common interface, allowing production hardware and simulation to be swapped without changing upstream code |
| **Pipeline (DAG of Stages)** | Processing pipeline modeled as a Directed Acyclic Graph. `PipelineBuilder` chains `AddStage()` calls. Each `Pipeline` has `StartStage`, `FinalStage`, a `GlobalBlackBoard`, and tracks context/state |
| **Stage abstraction** | `IStage` / `Stage<TInput, TOutput>` — generic base using TPL Dataflow blocks. Each stage defines input/output types, upstream/downstream linking, and configurable `MaxDegreeOfParallelism` |
| **Builder pattern** | `PipelineBuilder` constructs pipeline DAGs by chaining method calls |
| **Behavior Tree** | Tree-structured orchestration of algorithm execution within pipeline stages. Each stage has its own behavior tree with specialized node types (invokers, conditions, sequences) |
| **Blackboard pattern** | Shared data store for behavior tree nodes. `ConstBlackBoard` (long-lived/DCS-scope caches) and `GlobalBlackBoard` (per-pipeline transient data). Avoids parameter threading across tree nodes |
| **Singleton pattern** | `Singleton<T>` base class used by core orchestrators and data providers. Process-level mutex guard prevents duplicate instances |
| **God-class** | Main orchestrator classes (~2000+ lines) that initialize all subsystems, manage endpoints, subscribe to all events, and dispatch via large switch statements |
| **Command pattern** | `ICommand` interface for CLI operations. Each command encapsulates an action with execute semantics |
| **Service Worker pattern** | `ServiceWorker` abstract base — per-entity background worker with its own event queue (`NisQueue<T>`), rate limiter, and pub/sub wiring. Used in LCS for per-machine processing |
| **Paired service architecture** | Each domain has a `BackendService` (business logic, `IHostedService`) paired with an `APIService` (REST/event bridging). Clean separation of processing from API surface |
| **Repository pattern** | Data access abstracted behind `IRepository<T>` interfaces. Keyed singletons for different contexts (running vs judging) |
| **Object pooling** | `FamilyObjectPool` for native handle wrappers. Avoids repeated allocation/deallocation of expensive native resources |
| **Flyweight pattern** | Entity contexts support flyweight inflation — lightweight references expanded to full objects on demand, with dirty tracking for change detection |
| **EnvHolder (Thread-safe config)** | `EnvHolder.Create()` / `CreateOrGet()` / `Reset()` — thread-safe singleton holder for configuration objects, used pervasively across all services |

---

## Communication & Protocols

| Keyword | Detail |
|---|---|
| **MessagePack** | Binary serialization for all TCP messages. AOT-generated resolvers (code-gen via batch script) for Unity compatibility. Primary wire format between processes |
| **JSON (Newtonsoft + STJ)** | `Newtonsoft.Json` for configuration/logging/API payloads. `System.Text.Json` also used in newer code. Dual-stack |
| **TCP socket (custom framing)** | Raw TCP with custom message framing. `NetworkClient`/`NetworkServer`/`NetworkSession` abstractions. MessagePack payload with typed headers for routing |
| **REST API** | ASP.NET Core controllers with Swagger/OpenAPI (Swashbuckle). API versioning via `Asp.Versioning.Mvc` |
| **GraphQL** | `GraphQL.AspNet` — structured query API layer with typed queries, controllers, and repository data sources |
| **WebSocket** | Both server endpoints (ASP.NET middleware) and outbound client connections (`Websocket.Client`). Used for real-time streaming and monitoring |
| **SignalR** | Hub-based real-time push for web review features |
| **Named Pipes** | IPC between process supervisor and child processes |
| **RS-232 Serial** | `System.IO.Ports` for hardware communication |
| **STX/ETX framing** | Start/End of text byte framing for certain TCP protocols. UTF-8 message payload between delimiters |
| **Gzip compression** | Binary TCP messages compressed with gzip before transmission |
| **Memory-mapped files** | `PoolableMemoryMappedFile` / `MmfBuffer` for zero-copy large data exchange between processes |
| **Auto-reconnect** | TCP client drivers implement automatic reconnection with configurable retry logic |
| **Request/Response/Notice** | Message naming convention: `*Request` (ask), `*Response` (reply), `*Notice` (fire-and-forget notification). Request-tracking headers for correlation |

---

## Concurrency & Threading

| Keyword | Detail |
|---|---|
| **TPL Dataflow** | `System.Threading.Tasks.Dataflow` — backbone of the pipeline. `TransformManyBlock` (fan-out), `TransformBlock` (1:1), `ActionBlock` (terminal). Configurable `MaxDegreeOfParallelism` per stage |
| **ThreadPool tuning** | `ThreadPool.SetMinThreads` / `SetMaxThreads` configured from environment settings based on core count and machine type |
| **ConcurrentDictionary** | Primary thread-safe collection — session tracking, caches, latency maps, event subscriptions |
| **ConcurrentQueue** | Lock-free queue for inter-thread data passing |
| **SemaphoreSlim** | Used for serializing TCP sends (one-at-a-time guarantee on shared socket) |
| **Named Mutex (AutoMutex)** | Kernel-level named mutex for cross-thread hardware serialization |
| **Process Mutex** | `new Mutex(true, "ProcessName")` — prevents duplicate process instances |
| **ManualResetEventSlim** | Lightweight signal for synchronizing manual/step-through operations |
| **Interlocked.CompareExchange** | Lock-free pattern for thread-safe dispose and shutdown guards (compare-and-swap) |
| **Thread priorities** | Critical stages run at `ThreadPriority.Highest` (image capture) or `AboveNormal` (measurement) |
| **Process priority guard** | RealTime priority auto-downgraded to High to prevent OS starvation |
| **Concurrency policies** | Enum-driven policies (Low/Mid/High/Extreme) that compute per-stage `MaxDegreeOfParallelism` from core count and memory |
| **BackgroundTask / IHostedService** | ASP.NET hosted service lifecycle (`StartAsync`/`StopAsync`) for long-running background workers |
| **NisQueue\<T\>** | Per-service event queue ensuring ordered processing within a worker |
| **ConcurrencyRateLimitActionHandler** | Configurable rate limiter per service worker to throttle event processing |
| **CancellationTokenSource** | Standard .NET cooperative cancellation for graceful shutdown and timeout management |
| **Task.Run / Task.Factory.StartNew** | Used for fire-and-forget parallel operations (data collection, preloading, async event dispatch) |
| **Server GC** | Explicitly enabled (`<ServerGarbageCollection>true</ServerGarbageCollection>`) for throughput optimization on multi-core servers |
| **Custom TaskScheduler** | Optional `NisTaskScheduler` for specialized scheduling needs |

---

## Native Interop (C++ / Unmanaged)

| Keyword | Detail |
|---|---|
| **P/Invoke** | Primary managed-to-native bridge via `[DllImport]`. Calls into vision algorithm DLLs, infrastructure DLLs, Win32 API |
| **IntPtr handle wrapping** | Native resource handles (`IntPtr`) wrapped in managed classes with pooling and explicit disposal |
| **GC.KeepAlive()** | Prevents premature garbage collection of managed objects that hold native handles during long-running native calls |
| **PinnedBuffer / PinnedBufferMat** | Pin managed byte arrays in memory (`GCHandle.Alloc` with `Pinned`) so native code can safely read/write them |
| **unsafe code blocks** | `unsafe` context for direct pointer manipulation when interfacing with native memory |
| **NativeLibrary.Load()** | Explicit dynamic DLL loading at runtime for optional native dependencies |
| **Memory-mapped files** | `PoolableMemoryMappedFile` for sharing large buffers between managed and native processes without copying |
| **C++/CLI (deprecated)** | Former bridge layer between C# and C++. Replaced by direct P/Invoke for simplicity and performance |
| **SharedProjects (.shproj)** | MSBuild shared item projects that distribute native DLLs to consuming projects via `.projitems` imports |
| **TBB malloc** | Intel Threading Building Blocks scalable memory allocator — replaces default malloc for native code |
| **mimalloc** | Microsoft's compact general-purpose allocator — alternative native memory allocator |
| **Native alloc backend** | Configurable memory allocation backend for native code, switchable via P/Invoke call |

---

## Data Storage & Persistence

| Keyword | Detail |
|---|---|
| **SQLite** | Local embedded database for per-machine result storage (`Microsoft.Data.Sqlite`) |
| **MySQL (MySqlConnector)** | Production relational database. Supports table partitioning |
| **MSSQL** | Alternative relational database option. Data provider supports both MSSQL and MySQL |
| **DuckDB** | Embedded columnar analytics engine (`DuckDB.NET.Data.Full`) for OLAP queries |
| **Parquet** | Columnar file format (`Parquet.Net`) for analytical data export/storage. Used alongside DuckDB |
| **RocksDB** | Embedded key-value store (`RocksDbSharp`) for high-throughput data. Preloaded asynchronously at startup |
| **Redis + EasyCaching** | `StackExchange.Redis` with EasyCaching abstraction for distributed/in-memory caching layer |
| **DB partitioning** | Automatic table partitioning for time-series data |
| **LogicalServerType** | Database routing abstraction — maps logical roles (Management, Result, Analysis) to physical DB connections |
| **Entity Framework 6** | Used in legacy interop layer (not main codebase) |
| **Dapper** | Lightweight ORM used in interop layer |

---

## Web & API Frameworks

| Keyword | Detail |
|---|---|
| **ASP.NET Core 9** | Web host framework. `Microsoft.NET.Sdk.Web` with Kestrel server |
| **Kestrel multi-port** | Single ASP.NET host listening on multiple ports (HTTP, WebSocket, proxy) configured at startup |
| **Swagger / OpenAPI** | API documentation via Swashbuckle, auto-generated from controller attributes |
| **API versioning** | `Asp.Versioning.Mvc` for versioned REST endpoints |
| **Angular** | SPA frontend framework (`.esproj` projects) served by ASP.NET backend |
| **Unity 6** | 3D visualization frontend (game engine). Uses `MonoSingleton<T>` pattern, UniTask for async |
| **WPF** | Used for deployment coordinator UI (MVVM with ViewModels/Views) |
| **WinForms** | Used for standalone utility tool |
| **Blazor** | `Blazored.LocalStorage` referenced — partial Blazor support |
| **JWT** | `System.IdentityModel.Tokens.Jwt` + `BouncyCastle.Cryptography` for authentication |
| **MCP (Model Context Protocol)** | Embedded MCP server (ASP.NET-hosted, HTTP/SSE transport). Exposes internal tools with bearer token + IP allowlist auth |

---

## Scheduling & Background Processing

| Keyword | Detail |
|---|---|
| **Cronos** | Cron expression parser/scheduler for periodic tasks (data rollover, monitoring, session expiry) |
| **CronJobService** | Hosted service that evaluates Cronos expressions and fires scheduled work |
| **IHostedService** | ASP.NET Core background service lifecycle for long-running workers |
| **Timer-based functions** | Performance monitor timers, periodic operations via `System.Threading.Timer` |

---

## Logging, Diagnostics & Error Handling

| Keyword | Detail |
|---|---|
| **NLog** | Structured logging framework. Every class uses `NLog.LogManager.GetCurrentClassLogger()`. Config via `nlog.config` |
| **Tracert pattern** | `context.Tracert(response, logger)` — structured request/response tracing with configurable TraceIn/TraceOut verbosity |
| **Global crash handlers** | `AppDomain.UnhandledException` (fatal + flush), `TaskScheduler.UnobservedTaskException` (warn + observe), `FirstChanceException` (optional trace with `[ThreadStatic]` reentrance guard) |
| **PID file** | `.Pid` file written at startup — detects abnormal shutdown on next launch |
| **SuppressHelper.Run()** | Wraps non-critical operations in try-catch to prevent process crash from ancillary failures |
| **ResponseWaiter** | Timeout-guarded synchronous request-response bridge over async EventBus |
| **Rate limiting** | Per-IP rate limiting on error log API to prevent infinite error loop flooding |
| **Crash dump diagnostics** | `Microsoft.Diagnostics.Runtime` / `NETCore.Client` for crash dump analysis |
| **ScottPlot** | Chart generation library for diagnostic/performance visualizations |
| **Exception hierarchy** | Base exception -> Critical exception (catch site must transition system to error state) -> Subsystem-specific exceptions |

---

## Build System & Tooling

| Keyword | Detail |
|---|---|
| **MSBuild** | Build system. Single `.sln` solution. `Directory.Build.props` / `Directory.Build.targets` for global settings |
| **.NET 9 SDK** | Target framework for all C# projects. x64 platform |
| **Solution filters (.slnf)** | Partial build definitions — load only specific subsystem projects for faster IDE/build |
| **SharedProjects (.shproj)** | MSBuild shared item projects for native DLL distribution across consuming projects |
| **Batch build scripts** | Numbered `_1x_build_*.bat` scripts for full/prepare/backend/GUI builds |
| **Unity batch build** | `SuperUnityBuild` plugin for headless Unity builds from command line |
| **DVC** | Data Version Control for large binary test data / reference images — keeps large files out of git |
| **Coverity** | Static analysis for defect detection |
| **Lizard** | Code complexity analysis |
| **CppCheck** | C++ static analysis |
| **NSIS** | Installer packaging tool |
| **MessagePack AOT codegen** | Batch script generates AOT resolvers for MessagePack serialization (required for Unity IL2CPP) |
| **.editorconfig** | Code style enforcement across the solution |
| **LibGit2Sharp** | Managed Git operations for version tracking without shelling out to git CLI |

---

## Third-Party Libraries (Non-Domain)

| Keyword | Detail |
|---|---|
| **OpenCV 4.10** | Computer vision library (native + `OpenCvSharp` managed wrapper) |
| **Halcon** | Industrial machine vision library (requires hardware dongle) |
| **CUDA** | NVIDIA GPU acceleration for parallel compute |
| **Intel IPP** | Intel Performance Primitives — optimized signal/image processing |
| **ONNX Runtime** | ML inference engine for neural network models |
| **Google Protobuf** | Binary serialization for specific interop layer |
| **NPOI** | Excel file read/write (`.xlsx`) |
| **CommandLineParser** | CLI argument parsing library |
| **Cysharp UniTask** | Unity-optimized async/await library (allocation-free) |
| **Websocket.Client** | Outbound WebSocket client library |

---

## Optimization Techniques

### Memory Optimization

| Keyword | Detail |
|---|---|
| **Server GC** | `<ServerGarbageCollection>true</ServerGarbageCollection>` enabled on all executables (DCS, LCS, FCS, TCS). Throughput-oriented GC for multi-core |
| **GC.TryStartNoGCRegion** | Suppresses GC during latency-critical image acquisition. `GC.TryStartNoGCRegion(16GB)` in OCS driver, restored in `finally` with `GC.EndNoGCRegion()`. Feature-flagged via `env.Experimental` |
| **LOH Compaction** | `GCSettings.LargeObjectHeapCompactionMode = CompactOnce` then forced Gen2 collection. Used in DCS resource commands, LCS maintenance commands, and OLAP metrics |
| **GC Heap Count Tuning** | Reads `AppContext.GetData("System.GC.HeapCount")` and `DOTNET_GCHeapCount` env var to diagnose/configure GC heap count on multi-core machines |
| **Memory-Mapped File Pooling** | `PoolableMemoryMappedFile` extends `FamilyObjectPoolable` — pools MMF instances grouped by capacity family. Native side supports `cudaHostRegister` (GPU DMA pinning) and `VirtualLock` (OS page lock) |
| **Custom Native Memory Pool** | `NisKvMemoryPool` singleton implementing `IKvMemoryPool`. Wraps `ByteResourceController` with atomic stats, `shared_mutex`, and `concurrent_unordered_map` tracking |
| **Custom OpenCV Mat Allocator** | `NisMatAllocator` overrides `cv::MatAllocator` to route all `cv::Mat` allocations through `NisKvMemoryPool` instead of `malloc/free`. All image buffers are pooled |
| **mimalloc** | Microsoft's compact allocator integrated as alternative native malloc. `mimalloc_stats_print()` exposed via P/Invoke for diagnostics |
| **TBB malloc** | Intel TBB scalable allocator. `scalable_allocation_command(TBBMALLOC_CLEAN_ALL_BUFFERS)` for periodic cleanup |
| **NativeMemory\<T\>** | `ref struct` allocating via `Marshal.AllocHGlobal`, exposed as `Span<T>`. Manual lifetime, zero GC pressure |
| **RecyclableMemoryStream** | Pooled `MemoryStream` replacement used across 20+ files in networking, serialization, and response caching. Avoids LOH allocations from large byte arrays |
| **GC.KeepAlive()** | Prevents premature collection of managed objects holding native handles during long P/Invoke calls. Heavy use in inspection handler (10+ calls per method) |
| **EmptyWorkingSet** | P/Invoke to `psapi.dll!EmptyWorkingSet()` to trim process working set on command |
| **Flyweight pattern** | `FlyweightSmtPart` shares immutable template data across many inspection object instances, reducing per-object memory footprint |
| **Object pooling** | `FamilyObjectPool` / `ObjectPool<T>` with `Get()/Release()` pattern. Used for native handle wrappers, query builders, UI list items, memory-mapped files |

### CPU / Compute Optimization

| Keyword | Detail |
|---|---|
| **CUDA GPU processing** | `cudaHostAlloc`, `cudaMallocHost`, `cudaHostRegister` for pinned host memory. `cudaMalloc` for device memory. `cudaStream_t` for async GPU work. Custom GPU memory allocator and pinned memory allocator |
| **CUDA stream processing** | Stream-based 3D reconstruction with GPU memory pools and external GPU allocators for overlapping compute and data transfer |
| **ONNX Runtime + TensorRT** | `SessionOptions.MakeSessionOptionWithTensorrtProvider()` for GPU-accelerated ML inference, falling back to CPU. Sessions cached in `ConcurrentDictionary` to avoid repeated model loading |
| **Parallel.ForEach** | Pervasive use with `MaxDegreeOfParallelism` from `Env.ProcessorCount(concurrency)`. Used for image writes, file operations, data processing |
| **Thread priority per stage** | Each pipeline stage declares `StageThreadPriority`. Thread priority changed before processing, restored in `finally`. GrabStage = Highest, MeasureAndJudge = AboveNormal |

### I/O Optimization

| Keyword | Detail |
|---|---|
| **Memory-mapped file I/O** | Zero-copy data exchange between managed and native processes via `CreateFileMapping` / `MapViewOfFile`. Optionally CUDA-registered for GPU DMA |
| **GZip compression** | `System.IO.Compression.GZipStream` for OLAP writes and TCP message compression — reduces I/O bandwidth |
| **RecyclableMemoryStream** | Pooled stream buffers for serialization/network I/O to avoid repeated large allocations |

### Caching Strategies

| Keyword | Detail |
|---|---|
| **Redis distributed cache** | `StackExchange.Redis` with `EasyCaching` abstraction. Used for data provider cache, HTTP response caching middleware (`NisResponseCacheMiddleware`) |
| **In-memory ConcurrentDictionary caches** | Static `ConcurrentDictionary` caches for user/permission/role data, teaching data, computed lead data, ONNX sessions |
| **Connection pool warm-up** | Fire-and-forget `Task.Run` at startup executes dummy MySQL query to force connection pool creation. Logged with elapsed time |
| **Lazy\<T\> deferred init** | `Lazy<T>` for expensive registries, schema maps, label resolvers, driver managers. Defers construction cost until first access |
| **Preloaded data pass-through** | `preloadedJobInfo` parameter pattern — callers pass already-fetched data to downstream methods to avoid redundant async requests |

### Database Optimization

| Keyword | Detail |
|---|---|
| **DuckDB + Parquet OLAP** | Columnar analytics engine with day-partitioned Parquet storage (`day=YYYY-MM-DD` directories). Manifest tracking, separate read/write connection pools |
| **DuckDB connection pooling** | `DuckDbConnectionPool` with `pool.RentAsync()` checkout pattern. Separate pools for reads and writes. Pool size diagnostics via `SnapshotInFlight` |
| **MySQL HASH partitioning** | `PARTITION BY HASH(ContextId) PARTITIONS 32` on high-volume tables (parts, pads, materials, conditions) |
| **QueryBuilder object pooling** | `ObjectPool<PooledQueryBuilder>` pools `StringBuilder`-backed query builders with capacity limits. Avoids frequent allocation during query construction |
| **Time-based partition pruning** | `ArchiveCore()` scans `machine_id=*/day=*` directories, parses dates, deletes directories older than threshold. Automatic Parquet storage growth management |

### Pipeline / Throughput Optimization

| Keyword | Detail |
|---|---|
| **TPL Dataflow tuning** | Per-stage `ExecutionDataflowBlockOptions`: `MaxDegreeOfParallelism` from concurrency policy, `EnsureOrdered = false` (out-of-order for throughput), `SingleProducerConstrained = true` (optimization hint for single upstream) |
| **Stage slot ID queuing** | `ConcurrentQueue<int>` of slot IDs per multi-parallel stage — enables per-slot resource isolation without locking |
| **Bounded Channel backpressure** | `Channel.CreateBounded<Func<Task>>(capacity)` for OLAP analysis lane. Prevents unbounded memory growth under load |
| **Rate-limited concurrency** | `ConcurrencyRateLimitActionHandler` using `System.Threading.RateLimiting.RateLimitLease`. Controls parallel operation concurrency with WaitAll barriers |
| **Background service queue** | Configurable `Channel.CreateUnbounded<T>` or `Channel.CreateBounded<T>(capacity)` for background task queuing with backpressure |
| **SingleReader channel** | `BoundedChannelOptions { SingleReader = true }` enables lock-free reads on send queues — measurable throughput improvement for single-consumer patterns |

### Serialization Optimization

| Keyword | Detail |
|---|---|
| **MessagePack AOT codegen** | Source-generated formatters with `GetSpan_` methods. Avoids runtime reflection and JIT overhead. Required for Unity IL2CPP (no runtime emit) |
| **RecyclableMemoryStream** | Pooled buffers for MessagePack serialization — avoids LOH-triggering byte array allocations |
| **Native C++ msgpack** | `thirdparty/msgpack-cpp/` for native-layer serialization matching the managed wire format |

### Network Optimization

| Keyword | Detail |
|---|---|
| **TCP NoDelay** | `TcpClient.NoDelay = true` disables Nagle algorithm for low-latency responses on JSON/review connections |
| **TcpOption struct** | Configurable socket options: `ReuseAddress`, `KeepAlive`, `NoDelay`, `ExclusiveAddressUse`, `SendBufferSize`, `ReceiveBufferSize`, `TcpKeepAliveRetryCount/Time/Interval`. Defaults tuned per use case |
| **ExclusiveAddressUse** | `SocketOptionName.ExclusiveAddressUse = true` prevents port hijacking on server sockets |
| **Bounded send queue** | `SendFrame` struct holds `ArrayPool`-rented `byte[]` + length. Bounded channel prevents unbounded memory growth. Single background pump for serialized writes |

### Startup Optimization

| Keyword | Detail |
|---|---|
| **TieredCompilation** | `<TieredCompilation>true</TieredCompilation>` — JIT compiles methods at low quality first for fast startup, recompiles hot methods later |
| **ReadyToRun (R2R)** | Pre-compiled native code in assemblies — eliminates JIT cost on first call. Conditional on publish profile |
| **Connection pool pre-warming** | Fire-and-forget dummy queries at startup to force MySQL connection pool creation before real traffic arrives |
| **Lazy\<T\> deferred init** | Expensive registries, schema maps, and managers use `Lazy<T>` to defer construction until first access — reduces startup wall time |
