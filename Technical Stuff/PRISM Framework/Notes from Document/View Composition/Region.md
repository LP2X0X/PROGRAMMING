```ad-note
Regions are enabled in the Prism Library through a region manager, regions, and region adapters.
```
- In the Prism framework, a "region" refers to a region of a user interface that is defined for hosting and displaying views.
- A region is a class that implements the IRegion interface. The term region represents a container that can hold dynamic data that is presented in a UI.
- A region allows the Prism Library to place dynamic content contained in modules in predefined placeholders in a UI container.
- Regions can hold any type of UI content. A module can contain UI content presented as a user control, a data type that is associated with a data template, a custom control, or any combination of these. This lets you define the appearance for the UI areas and then have modules place content in these predetermined areas.
```ad-note
A region can contain zero or more items.
```
![[Ch7UIFig3.png|center]]

---
- Prism regions are essentially named placeholders within which views can be displayed. Any control in the application's UI can be a declared a region by simply adding a **RegionName** attached property to it, as shown here.
```xml
<ContentControl prism:RegionManager.RegionName="MainRegion" ... />
```
- For each control specified as a region, Prism creates a Region object to represent the region and a RegionAdapter object, which manages the placement and activation of views into the specified control.
- The Prism Library provides RegionAdapter implementations for most of the common WPF controls. You can create a custom RegionAdapter to support additional controls or when you need to define a custom behavior. The RegionManager class provides access to the Region objects within the application.
- In many cases, the region control will be a simple control, such as a ContentControl, that can display one view at a time. In other cases, the Region control will be a control that is able to display multiple views at the same time, such as a TabControl or a ListBox control.
- The region adapter manages the active state of the views within the region. The active view is the view that is the selected or top-most view—for example, in a TabControl, the active view is the one displayed in the selected tab; in a ContentControl, the active view is the view that is currently displayed as the control's content.