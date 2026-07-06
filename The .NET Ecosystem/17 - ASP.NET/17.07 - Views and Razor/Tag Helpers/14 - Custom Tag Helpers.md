---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


You can create your own tag helpers by inheriting from `TagHelper` and overriding `Process()` or `ProcessAsync()`.

### Basic Structure

```csharp
using Microsoft.AspNetCore.Razor.TagHelpers;

[HtmlTargetElement("alert")]  // Targets <alert> elements
public class AlertTagHelper : TagHelper
{
    public string Type { get; set; } = "info";  // Maps to the "type" attribute

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        // Change the HTML element from <alert> to <div>
        output.TagName = "div";

        // Map our type to Bootstrap classes
        var cssClass = Type.ToLower() switch
        {
            "warning" => "alert alert-warning",
            "danger" or "error" => "alert alert-danger",
            "success" => "alert alert-success",
            _ => "alert alert-info"
        };

        output.Attributes.SetAttribute("class", cssClass);
        output.Attributes.SetAttribute("role", "alert");

        // The child content of <alert>...</alert> is preserved automatically
    }
}
```

Usage:

```cshtml
<alert type="warning">
    <strong>Warning!</strong> Your subscription expires in 3 days.
</alert>
```

Renders:

```html
<div class="alert alert-warning" role="alert">
    <strong>Warning!</strong> Your subscription expires in 3 days.
</div>
```

### Targeting Existing HTML Elements

Instead of creating a custom element, you can target existing HTML elements with specific attributes:

```csharp
[HtmlTargetElement("a", Attributes = "asp-confirm")]
public class ConfirmLinkTagHelper : TagHelper
{
    [HtmlAttributeName("asp-confirm")]
    public string ConfirmMessage { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.Attributes.SetAttribute("onclick",
            $"return confirm('{ConfirmMessage}');");
        output.Attributes.RemoveAll("asp-confirm");
    }
}
```

Usage:

```cshtml
<a asp-controller="Products" asp-action="Delete"
   asp-route-id="@product.Id"
   asp-confirm="Are you sure you want to delete this product?">
    Delete
</a>
```

### Async Tag Helpers

For tag helpers that need to read child content:

```csharp
[HtmlTargetElement("markdown")]
public class MarkdownTagHelper : TagHelper
{
    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        var childContent = await output.GetChildContentAsync();
        var markdownText = childContent.GetContent();

        // Convert markdown to HTML (using a library like Markdig)
        var html = Markdig.Markdown.ToHtml(markdownText);

        output.TagName = "div";
        output.Attributes.SetAttribute("class", "markdown-content");
        output.Content.SetHtmlContent(html);
    }
}
```

### The [HtmlTargetElement] Attribute

| Property | Purpose | Example |
|---|---|---|
| Tag name (positional) | Which element to target | `[HtmlTargetElement("alert")]` |
| `Attributes` | Only target elements with these attributes | `[HtmlTargetElement("a", Attributes = "asp-confirm")]` |
| `TagStructure` | Self-closing or with content | `TagStructure.NormalOrSelfClosing` |
| `ParentTag` | Only target when inside a specific parent | `[HtmlTargetElement("li", ParentTag = "ul")]` |

> [!warning] Common Misconception
> Tag helper properties are mapped from HTML attributes using **kebab-case to PascalCase** conversion. A C# property `MaxItems` is set via the attribute `max-items`. You can override this with `[HtmlAttributeName("custom-name")]`.

> [!summary] Section Summary
> - Custom tag helpers inherit from `TagHelper` and override `Process()` or `ProcessAsync()`
> - `[HtmlTargetElement]` specifies which element(s) the tag helper targets
> - Tag helper properties automatically map from kebab-case HTML attributes to PascalCase C# properties
> - You can create new elements (`<alert>`) or enhance existing ones (`<a asp-confirm="...">`)
> - `ProcessAsync` is needed when reading child content

---
