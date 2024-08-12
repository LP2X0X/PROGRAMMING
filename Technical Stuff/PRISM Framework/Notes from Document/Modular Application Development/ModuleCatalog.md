## Definition
- The `ModuleCatalog` holds information about the [[Module|modules]] that can be used by the application. The catalog is essentially a collection of `ModuleInfo` classes.
## Additional Info
- The module catalog is represented by a class that implements the `IModuleCatalog` interface. The module catalog class is **created by** the `PrismApplication` base class **during application initialization**.
## How to use
- There are several typical approaches to filling the **ModuleCatalog** with **ModuleInfo** instances:
	- Registering modules in code
	- Registering modules in XAML
	- Registering modules in a configuration file
	- Discovering modules in a local directory on disk
