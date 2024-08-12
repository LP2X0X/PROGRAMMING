![[Pasted image 20240118144513.png|center]]
# View
- The view is a visual element, such as a window, page, user control, or data template. The view defines the controls contained in the view and their visual layout and styling.
- The view references the view model through its **DataContext** property. The controls in the view are data bound to the properties and commands exposed by the view model.
- The view may customize the data binding behavior between the view and the view model. For example, the view may use value converters to format the data to be displayed in the UI, or it may use validation rules to provide additional input data validation to the user.
- The view defines and handles UI visual behavior, such as animations or transitions that may be triggered from a state change in the view model or via the user's interaction with the UI.
- The view's code-behind may define UI logic to implement visual behavior that is difficult to express in XAML or that requires direct references to the specific UI controls defined in the view.
# View Model
- The view model is a non-visual class and does not derive from any WPF base class. It encapsulates the presentation logic required to support a use case or user task in the application. The view model is testable independently of the view and the model.
- The view model typically does not directly reference the view. It implements properties and commands to which the view can data bind. It notifies the view of any state changes via change notification events via the **INotifyPropertyChanged** and **INotifyCollectionChanged** interfaces.
- The view model coordinates the view's interaction with the model. It may convert or manipulate data so that it can be easily consumed by the view and may implement additional properties that may not be present on the model. It may also implement data validation via the **IDataErrorInfo** or **INotifyDataErrorInfo** interfaces.
- The view model may define logical states that the view can represent visually to the user.
# Model
- Model classes are non-visual classes that encapsulate the application's data and business logic. They are responsible for managing the application's data and for ensuring its consistency and validity by encapsulating the required business rules and data validation logic.
- The model classes do not directly reference the view or view model classes and have no dependency on how they are implemented.
- The model classes typically provide property and collection change notification events through the **INotifyPropertyChanged** and **INotifyCollectionChanged** interfaces. This allows them to be easily data bound in the view. Model classes that represent collections of objects typically derive from the **ObservableCollection<T>** class.
- The model classes typically provide data validation and error reporting through either the **IDataErrorInfo** or **INotifyDataErrorInfo** interfaces.
- The model classes are typically used in conjunction with a service or repository that encapsulates data access and caching.