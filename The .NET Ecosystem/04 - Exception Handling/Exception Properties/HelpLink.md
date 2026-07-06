---
tags:
 - csharp
 - exception-handling
---

## `Exception.HelpLink`

The `HelpLink` property holds a URL or URN pointing to a help resource that explains the error in detail. While `Message` tells the user *what* went wrong, `HelpLink` tells them *where to learn more*.

### Basic Usage

```csharp
throw new InvalidOperationException("License has expired.")
{
    HelpLink = "https://docs.example.com/errors/license-expired"
};
```

### Reading HelpLink in a Handler

```csharp
catch (Exception ex)
{
    Console.WriteLine(ex.Message);

    if (!string.IsNullOrEmpty(ex.HelpLink))
    {
        Console.WriteLine($"For more info, see: {ex.HelpLink}");
    }
}
```

### Tips & Best Practices

- **Default value is an empty string**, not `null`. Always check with `string.IsNullOrEmpty()` rather than a null check alone.
- **Set it on custom exceptions.** If you define domain-specific exceptions, set `HelpLink` in the constructor so every throw site gets it automatically:
  ```csharp
  public class LicenseExpiredException : Exception
  {
      public LicenseExpiredException()
          : base("Your license has expired.")
      {
          HelpLink = "https://docs.example.com/errors/license-expired";
      }
  }
  ```
- **Use stable, versioned URLs.** Link to documentation that won't move. Include a version or error code in the URL (e.g., `/errors/E1042`) so the page stays relevant even as the product evolves.
- **It can also point to local help files** using a URN or file path (e.g., `"file:///C:/Help/errors.chm"`), though URLs are far more common in modern applications.
- **Useful for APIs and libraries.** When building libraries consumed by other developers, `HelpLink` pointing to your docs dramatically improves the developer experience compared to a bare message string.
- **Logging frameworks can capture it.** Tools like Serilog and Application Insights will pick up `HelpLink` automatically, making it searchable in your dashboards.
