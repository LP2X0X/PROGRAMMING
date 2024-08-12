- In state-based navigation, the view that represents the UI is updated either through state changes in the view model or through the user's interaction within the view itself. In this style of navigation, instead of replacing the view with another view, the view's state is changed. Depending on how the view's state is changed, the updated UI may feel to the user like navigation.
- This style of navigation is suitable in the following situations:
    - The view needs to display the same data or functionality in different styles or formats.
    - The view needs to change its layout or style based on the underlying state of the view model.
    - The view needs to initiate limited modal or non-modal interaction with the user within the context of the view.
## Displaying Data in Different Formats or Styles
- Because the view is presenting the same data, but in a different visual representation, the view model is not required to be involved in the navigation between representations. In this case, navigation is entirely handled within the view itself.