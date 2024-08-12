![[ModularityDi.png|center]]
In this example, the `OrdersModule` assembly defines an `OrdersRepository` class (along with other views and classes that implement order functionality).
The `CustomerModule` assembly defines a `CustomersViewModel` class which depends on the `OrdersRepository`, typically based on an interface exposed by the service.
The application startup and bootstrapping process contains the following steps:
1. The `App` class that is derived from `PrismApplication` starts the module initialization process, and the module loader loads and initializes the `OrdersModule`.
2. In the initialization of the `OrdersModule`, it registers the `OrdersRepository` with the container.
3. The module loader then loads the `CustomersModule`. **The order of module loading can be specified by the dependencies in the module metadata.**
4. The `CustomersModule` constructs an instance of the `CustomerViewModel` by resolving it through the container. The `CustomerViewModel` has a dependency on the `OrdersRepository` (typically based on its interface) and indicates it through constructor or property injection. The container injects that dependency in the construction of the view model based on the type registered by the `OrdersModule`. The net result is an interface reference from the `CustomerViewModel` to the `OrderRepository` without tight coupling between those classes.