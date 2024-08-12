## Definition
- A module is a logical collection of functionality and resources that is packaged in a way that can be separately developed, tested, deployed, and integrated into an application. Modules are a key concept in Prism's modular architecture, allowing developers to design and implement loosely-coupled and independently deployable parts of an application.
- It can include a collection of related components, such as application features, including user interface and business logic, or pieces of application infrastructure, such as application-level services for logging or authenticating users.
- Modules are independent of one another but can communicate with each other in a loosely coupled fashion.
## Life cycle
The module loading process in Prism includes the following sequence:
- **Registering**: Modules are created by implementing the IModule interface inside of a class.
- **Discovering modules**: The modules to be loaded at run-time for a particular application are defined in a [[ModuleCatalog|Module catalog]]. The catalog contains information about the modules to be loaded, such as their location, and the order in which they are to be loaded.
- **Loading modules**: The assemblies that contain the modules are loaded into memory.
- **Initializing modules**: The modules are then initialized. This means creating instances of the module class and calling the `RegisterTypes` and `OnInitialized` methods on them via the `IModule` interface.
## When to load a module
Prism applications can initialize modules as soon as possible, known as "when available," or when the application needs them, known as "on-demand." Consider the following guidelines for loading modules:
- Modules required for the application to run must be loaded with the application and initialized when the application runs.
- Modules containing features that are rarely used (or are support modules that other modules optionally depend upon) can be loaded and initialized on-demand.
## Communicate between modules
Even though modules should have low coupling between each other, it is common for modules to communicate with each other. There are several loosely coupled communication patterns, each with their own strengths. Typically, combinations of these patterns are used to create the resulting solution. The following are some of these patterns:
- **Loosely coupled events**. A module can broadcast that a certain event has occurred. Other modules can subscribe to those events so they will be notified when the event occurs. Loosely coupled events are a lightweight manner of setting up communication between two modules; therefore, they are easily implemented. However, a design that relies too heavily on events can become hard to maintain, especially if many events have to be orchestrated together to fulfill a single task. In that case, it might be better to consider a shared service.
- **Shared services**. A shared service is a class that can be accessed through a common interface. Typically, shared services are found in a shared assembly and provide system-wide services, such as authentication, logging, or configuration.
- **Shared resources**. If you do not want modules to directly communicate with each other, you can also have them communicate indirectly through a shared resource, such as a database or a set of web services.