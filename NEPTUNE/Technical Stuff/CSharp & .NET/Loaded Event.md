- Loaded event is an event in FrameworkElement class which occurs when the element is laid out, rendered, and ready for interaction.
```csharp
public MainWindow()
{
	this.Loaded += MainWindow_Loaded;
}

private void MainWindow_Loaded(object sender, RoutedEventArgs e)
{
	// Do something here when the mainwindow is loaded
}
```
