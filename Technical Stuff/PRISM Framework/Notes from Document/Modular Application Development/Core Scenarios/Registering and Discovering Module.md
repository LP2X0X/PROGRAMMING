- The modules that an application can load are defined in a [[ModuleCatalog|module catalog]]. The Prism Module Loader uses the module catalog to determine which modules are available to be loaded into the application, when to load them, and in which order they are to be loaded.
## Registering
### 1 - Registering in Code
- To register the module directly with the `ModuleCatalog` class, call the `AddModule` method in your application's `PrismApplication` derived `App` class. Override `ConfigureModuleCatalog` to add your modules:
```csharp
protected override void ConfigureModuleCatalog()
{
    Type moduleCType = typeof(ModuleC);
    ModuleCatalog.AddModule(new ModuleInfo()
    {
        ModuleName = moduleCType.Name,
        ModuleType = moduleCType.AssemblyQualifiedName,
        InitializationMode = InitializationMode.OnDemand, // Change ini mode (on demand or when available)
    });
}
```
```ad-note
If your application has a direct reference to the module type, you can add it by type as shown above; otherwise you need to provide the fully qualified type name and the location of the assembly.
```
- Here we use attribute:
```csharp
[Module(ModuleName = "ModuleA")]
[ModuleDependency("ModuleD")]
public class ModuleA : IModule
{
    ...
}
```
### 2 - Registering in XAML
- When registering by this method, the module catalog is created by the App with a call to the `CreateFromXaml` method:
```csharp
protected override IModuleCatalog CreateModuleCatalog()
{
    return ModuleCatalog.CreateFromXaml(new Uri("/MyProject;component/ModulesCatalog.xaml", UriKind.Relative));
}
```

```xml
<--! ModulesCatalog.xaml -->
<Modularity:ModuleCatalog xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:sys="clr-namespace:System;assembly=mscorlib"
    xmlns:Modularity="clr-namespace:Microsoft.Practices.Prism.Modularity;assembly=Microsoft.Practices.Prism">

    <Modularity:ModuleInfoGroup Ref="file://DirectoryModules/ModularityWithMef.Desktop.ModuleB.dll" InitializationMode="WhenAvailable">
        <Modularity:ModuleInfo ModuleName="ModuleB" ModuleType="ModularityWithMef.Desktop.ModuleB, ModularityWithMef.Desktop.ModuleB, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    </Modularity:ModuleInfoGroup>

    <Modularity:ModuleInfoGroup InitializationMode="OnDemand">
        <Modularity:ModuleInfo Ref="file://ModularityWithMef.Desktop.ModuleE.dll" ModuleName="ModuleE" ModuleType="ModularityWithMef.Desktop.ModuleE, ModularityWithMef.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
        <Modularity:ModuleInfo Ref="file://ModularityWithMef.Desktop.ModuleF.dll" ModuleName="ModuleF" ModuleType="ModularityWithMef.Desktop.ModuleF, ModularityWithMef.Desktop.ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null">
            <Modularity:ModuleInfo.DependsOn>
                <sys:String>ModuleE</sys:String>
            </Modularity:ModuleInfo.DependsOn>
        </Modularity:ModuleInfo>
    </Modularity:ModuleInfoGroup>

    <!-- Module info without a group -->
    <Modularity:ModuleInfo Ref="file://DirectoryModules/ModularityWithMef.Desktop.ModuleD.dll" ModuleName="ModuleD" ModuleType="ModularityWithMef.Desktop.ModuleD, ModularityWithMef.Desktop.ModuleD, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
</Modularity:ModuleCatalog>
```
### 3 - Registering in Configuration File
- In WPF, it is possible to specify the module information in the **App.config file**.
- The advantage of this approach is that this file is **not compiled into the application**. This makes it very easy to add or remove modules at run time without recompiling the application.
```xml
<!-- ModularityWithUnity.Desktop\\app.config -->
<xml version="1.0" encoding="utf-8" ?>
<configuration>
    <configSections>
        <section name="modules" type="Prism.Modularity.ModulesConfigurationSection, Prism.Wpf"/>
    </configSections>

    <modules>
        <module assemblyFile="ModularityWithUnity.Desktop.ModuleE.dll" moduleType="ModularityWithUnity.Desktop.ModuleE, ModularityWithUnity.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" moduleName="ModuleE" startupLoaded="false" />
        <module assemblyFile="ModularityWithUnity.Desktop.ModuleF.dll" moduleType="ModularityWithUnity.Desktop.ModuleF, ModularityWithUnity.Desktop.ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" moduleName="ModuleF" startupLoaded="false">
            <dependencies>
                <dependency moduleName="ModuleE"/>
            </dependencies>
        </module>
    </modules>
</configuration>
```
- In your application's `App` class, you need to specify that the configuration file is the source for your `ModuleCatalog`. To do this, override the `CreateModuleCatalog` method and return an instance of the `ConfigurationModuleCatalog` class.
```csharp
protected override IModuleCatalog CreateModuleCatalog()
{
    return new ConfigurationModuleCatalog();
}
```
## Discovering
### Discovering in Directory
- The Prism `DirectoryModuleCatalog` class allows you to specify a local directory as a module catalog in WPF. This module catalog will scan the specified folder and search for assemblies that define the modules for your application.
- To use this approach, you will need to use declarative attributes on your module classes to specify the module name and any dependencies that they have.
- In App.xaml.cs:
```csharp
protected override IModuleCatalog CreateModuleCatalog()
{
    return new DirectoryModuleCatalog() {ModulePath = @".\\Modules"};
}
```
- In the module cs file:
```csharp
[Module(ModuleName = "ModuleA", OnDemand = true)]
[ModuleDependency("ModuleD")]
public class ModuleA : IModule
{
    ...
}
```