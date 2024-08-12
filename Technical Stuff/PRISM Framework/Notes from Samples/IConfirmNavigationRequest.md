```csharp
    public class ViewAViewModel : BindableBase, IConfirmNavigationRequest
    {
        public ViewAViewModel()
        {
            SwitchCommand = new DelegateCommand(Switch, CanSwitch);
        }
		
        private bool CanSwitch()
        {
            return true;
        }
		
        private void Switch()
        {
            navigationCallback(true);
        }

        public DelegateCommand SwitchCommand { get; private set; }

        private Action<bool> navigationCallback;

        public void ConfirmNavigationRequest(NavigationContext navigationContext, Action<bool> continuationCallback)
        {
            bool result = true;

            if (MessageBox.Show("Do you to navigate?", "Navigate?", MessageBoxButton.YesNo) == MessageBoxResult.No)
                result = false;

            navigationCallback = continuationCallback;
        }

        public bool IsNavigationTarget(NavigationContext navigationContext)
        {
            return true;
        }

        public void OnNavigatedFrom(NavigationContext navigationContext)
        {
            
        }

        public void OnNavigatedTo(NavigationContext navigationContext)
        {
            
        }
    }
}

```

```ad-note
You can call continuationCallback directly in the ConfirmNavigationRequest or store it like the above example.
```