## 1 - Region Manager
- The RegionManager class is responsible for creating and maintaining a collection of regions for the host controls. The RegionManager uses a control-specific adapter that associates a new region with the host control.
![[Ch7UIFig2.png|center]]
- The RegionManager can create regions in code or in XAML. The **RegionManager.RegionName attached property** is used to create a region in XAML by applying the attached property to the host control.
- Applications can contain one or more instances of a RegionManager. You can specify the RegionManager instance into which you want to register the region. This is useful if you want to move the control around in the visual tree and do not want the region to be cleared when the attached property value is removed.
- The RegionManager provides a RegionContext attached property that permits its regions to share data.
## 2 - Region Adapter
```ad-important
To expose a UI control as a region, it must have a **region adapter**.
```
- Region adapters are responsible for creating a region and associating it with the control. This allows you to use the IRegion interface to manage the UI control contents in a consistent way.
- Each region adapter adapts a specific type of UI control. The Prism Library provides the following three region adapters:
    - ContentControlRegionAdapter. This adapter adapts controls of type System.Windows.Controls.ContentControl and derived classes.
    - SelectorRegionAdapter. This adapter adapts controls derived from the class System.Windows.Controls.Primitives.Selector, such as the System.Windows.Controls.TabControl control.
    - ItemsControlRegionAdapter. This adapter adapts controls of type System.Windows.Controls.ItemsControl and derived classes.
## 3 - Region Behaviors
- A region behavior is a class that is attached to a region to give the region additional functionality. This behavior is attached to the region and remains active for the lifetime of the region.

