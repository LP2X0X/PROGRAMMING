In WPF (Windows Presentation Foundation), a `Uri` (Uniform Resource Identifier) is a class used to represent a Uniform Resource Identifier, which can be a reference to a resource such as an image, media file, or other content. `Uri` is often used in the context of specifying the source or location of resources in XAML markup or in code.

Here are some common scenarios where `Uri` is used in WPF:

1. **Setting Source for Images or Media:**
   ```xaml
   <Image Source="pack://application:,,,/Images/myImage.png" />
   ```
   In this example, the `Uri` specifies the source for an image using the `pack` URI scheme.

2. **Setting Source for Hyperlinks:**
   ```xaml
   <Hyperlink NavigateUri="http://www.example.com">Visit our website</Hyperlink>
   ```
   Here, the `NavigateUri` property of the `Hyperlink` control is set using a `Uri` to define the target URL.

3. **Referencing External XAML Files:**
   ```xaml
   <Frame Source="Page.xaml" />
   ```
   The `Source` property of the `Frame` control can be set using a `Uri` to reference an external XAML file.

4. **Loading XAML dynamically in Code-Behind:**
   ```csharp
   Uri uri = new Uri("Page.xaml", UriKind.Relative);
   StreamResourceInfo info = Application.GetContentStream(uri);
   using (StreamReader reader = new StreamReader(info.Stream))
   {
       string xamlContent = reader.ReadToEnd();
       // Process the XAML content as needed
   }
   ```
   This example demonstrates using a `Uri` to load XAML content dynamically in code-behind.

`Uri` provides a flexible way to reference resources, whether they are located within the application's assembly, in external files, or on the web. The `UriKind` enumeration is used to specify the kind of the `Uri`, such as whether it is absolute or relative. The `Uri` class in WPF is part of the `System` namespace, specifically `System.Windows`.