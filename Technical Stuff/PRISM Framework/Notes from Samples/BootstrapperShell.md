```csharp
using Unity;
using Prism.Unity;
using BootstrapperShell.Views;
using System.Windows;
using Prism.Ioc;
using System.Runtime.CompilerServices;

namespace BootstrapperShell
{
    class Bootstrapper : PrismBootstrapper
    {
        protected override DependencyObject CreateShell()
        {
            return Container.Resolve<MainWindow>();
        }

        protected override void RegisterTypes(IContainerRegistry containerRegistry)
        {
            
        }
        protected override void InitializeShell(DependencyObject Shell)
        {
            Application.Current.MainWindow = (Window)Shell;
            Application.Current.MainWindow.Show();
        }

    }
}
```