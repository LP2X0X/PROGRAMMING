- You can register a mapping for a ViewModel to a specific view directly with the ViewModelLocator by using the ViewModelLocationProvider.Register method.
- The following examples show the various ways to create a mapping between a view called MainWindow and a ViewModel named CustomViewModel:

1 - Type / Type
```csharp
ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), typeof(CustomViewModel));
```
2 - Type / Factory
```csharp
ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), () => Container.Resolve<CustomViewModel>());
```
3 - Generic Factory
```csharp
ViewModelLocationProvider.Register<MainWindow>(() => Container.Resolve<CustomViewModel>());
```
4 - Generic Type
```csharp
ViewModelLocationProvider.Register<MainWindow, CustomViewModel>();
```
```ad-note
Registering your ViewModels directly with the ViewModelLocator is faster than relying on the default naming convention. This is because the naming convention requires the use of reflection, while a custom mapping provides the type directly to the ViewModelLocator.
```