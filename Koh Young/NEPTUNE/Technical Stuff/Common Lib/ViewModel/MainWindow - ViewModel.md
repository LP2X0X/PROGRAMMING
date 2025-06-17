- How to bind view and viewmodel:
```xml
<Window.DataContext>
	<local:MainViewViewModel />
</Window.DataContext>
<Window.Resources>
	<DataTemplate DataType="{x:Type vm:ResultParserViewViewModel}">
		<view:ResultParserView />
	</DataTemplate>
	<DataTemplate DataType="{x:Type vm:ImageViewerViewViewModel}">
		<view:ImageViewerView />
	</DataTemplate>
	<DataTemplate DataType="{x:Type vm:TrendChartViewViewModel}">
		<view:TrendChartView />
	</DataTemplate>
	<DataTemplate DataType="{x:Type vm:ChartViewViewModel}">
		<view:ChartView />
	</DataTemplate>
	<DataTemplate DataType="{x:Type vm:CompositionViewModel}">
		<view:CompositionView />
	</DataTemplate>
	<Style x:Key="SectionBorderStyle" TargetType="Border">
		<Setter Property="BorderBrush" Value="Transparent" />
		<Setter Property="BorderThickness" Value="1" />
		<!--  다른 Border 속성들을 여기에 추가할 수 있습니다.  -->
	</Style>
</Window.Resources>
```

---

- Switching between views:
```csharp
// The code use for switching
private void ChangeContentView(ViewSection viewType)
{
	this.CurrentViewModel = contentViewModel.FirstOrDefault(x => x.ViewType == viewType);
	this.CurrentViewModel?.UpdateBoard(inspBaordCond);
	this.CurrentViewModel?.UpdateItemResult(inspResult);
	this.ViewType = CurrentViewModel.ViewType;
	this.ChartViewModelUpdateItem();
}

// In the constructor, use prism's delegate command and create the view model object here
this.contentViewModel = new List<ISectionViewModel>()
{
	new ResultParserViewViewModel(),
	new ImageViewerViewViewModel(),
	new TrendChartViewViewModel(),
	new ChartViewViewModel(),
	new CompositionViewModel(),
};

this.ResultParserContentCommand = new DelegateCommand(() => ChangeContentView(ViewSection.ResultParser));
this.ImageViewerContentCommand = new DelegateCommand(() => ChangeContentView(ViewSection.ImageViewer));
this.ChartContentCommand = new DelegateCommand(() => ChangeContentView(ViewSection.ChartView));
this.TrendChartContentCommand = new DelegateCommand(() => ChangeContentView(ViewSection.TrendChartView));
this.CompositionContentCommand = new DelegateCommand(() => ChangeContentView(ViewSection.CompositionView));

// View section enum (In TestWindow.Model)
public enum ViewSection
{
	ResultParser,
	ImageViewer,
	TrendChartView,
	ChartView,
	CompositionView
}
```

```ad-note
 In each viewmodel class will have a public property name which implement a get function which will return the corresponding ViewSection value:
 ````csharp
 // In ImageViewerViewModel
 public ViewSection ViewType => ViewSection.ImageViewer; 
````

---
- Set here:
![[Pasted image 20240130101953.png|center]]
- So don't have to get again here:
![[Pasted image 20240130102025.png|center]]
