- ViewModel class:
```csharp
using Prism.Mvvm;

namespace ViewModelLocator.ViewModels // ViewModels namespace
{
    public class MainWindowViewModel : BindableBase
    {
        private string _title = "Prism Unity Application";
        public string Title
        {
            get { return _title; }
            set { SetProperty(ref _title, value); }
        }

        public MainWindowViewModel()
        {

        }
    }
}
```

^ae7e4c

- View class:
```csharp
using System.Windows;

namespace ViewModelLocator.Views // View namespace
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }
    }
}
```

^1c0bb3
- Same assembly:
![[Pasted image 20240116131422.png|center]] ^e61d6d