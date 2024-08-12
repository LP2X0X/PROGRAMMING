When using `INotifyPropertyChanged`, the UI control that is data-bound to a property will handle the `PropertyChanged` event. The purpose of implementing `INotifyPropertyChanged` is to notify subscribers (like UI controls) about changes to the properties of an object.

Here's a step-by-step explanation:

1. **Object with `INotifyPropertyChanged`:**
   - You have a class that implements `INotifyPropertyChanged`, and it has properties whose changes you want to notify the UI about.

    ```csharp
    public class Person : INotifyPropertyChanged
    {
        private string name;

        public string Name
        {
            get { return name; }
            set
            {
                if (name != value)
                {
                    name = value;
                    OnPropertyChanged(nameof(Name));
                }
            }
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged(string propertyName)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
    ```

2. **Data Binding in XAML:**
   - You bind an instance of this class to a UI control in XAML, specifying which property to bind to.

    ```xaml
    <TextBox Text="{Binding Name}" />
    ```

3. **UI Control Listens to Changes:**
   - The UI control (in this case, a `TextBox`) listens to changes in the bound property (`Name`) through data binding. When the property changes, the control automatically updates its display.

4. **PropertyChanged Event Handling:**
   - Internally, the `TextBox` (or any other control) subscribes to the `PropertyChanged` event of the data-bound object (an instance of `Person` in this case). When the `Name` property changes, the `PropertyChanged` event is raised, and the event handler in the UI control is triggered.

   The UI control, upon receiving the `PropertyChanged` event, updates its display with the new value of the property (`Name` in this case).

This mechanism enables a dynamic and automatic update of the UI when the underlying data model (the `Person` object) changes. It's a key feature in frameworks like WPF, UWP, and Xamarin, where data-binding is a fundamental part of the UI development process.