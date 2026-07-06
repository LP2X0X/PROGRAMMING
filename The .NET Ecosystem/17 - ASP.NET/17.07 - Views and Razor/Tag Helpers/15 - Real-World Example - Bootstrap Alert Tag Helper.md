---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


A more complete custom tag helper with dismissibility, icons, and a heading:

```csharp
using Microsoft.AspNetCore.Razor.TagHelpers;

[HtmlTargetElement("alert")]
public class AlertTagHelper : TagHelper
{
    /// <summary>
    /// Bootstrap alert type: primary, secondary, success, danger, warning, info, light, dark
    /// </summary>
    public string Type { get; set; } = "info";

    /// <summary>
    /// If true, adds a dismiss button to the alert.
    /// </summary>
    public bool Dismissible { get; set; } = false;

    /// <summary>
    /// Optional heading displayed in bold above the message.
    /// </summary>
    public string Heading { get; set; }

    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "div";

        var cssClass = $"alert alert-{Type}";
        if (Dismissible)
        {
            cssClass += " alert-dismissible fade show";
        }

        output.Attributes.SetAttribute("class", cssClass);
        output.Attributes.SetAttribute("role", "alert");

        // Build inner content
        if (!string.IsNullOrEmpty(Heading))
        {
            output.PreContent.SetHtmlContent($"<h4 class=\"alert-heading\">{Heading}</h4>");
        }

        // Child content is preserved automatically between PreContent and PostContent

        if (Dismissible)
        {
            output.PostContent.SetHtmlContent(
                "<button type=\"button\" class=\"btn-close\" data-bs-dismiss=\"alert\" aria-label=\"Close\"></button>");
        }
    }
}
```

Usage:

```cshtml
@* Simple alert *@
<alert type="success">Product saved successfully.</alert>

@* Dismissible alert with heading *@
<alert type="warning" heading="Subscription Expiring" dismissible="true">
    Your subscription expires on <strong>@Model.ExpirationDate.ToString("MMMM dd, yyyy")</strong>.
    <a href="/account/renew" class="alert-link">Renew now</a> to avoid interruption.
</alert>

@* Error alert *@
<alert type="danger" heading="Error">
    @foreach (var error in Model.Errors)
    {
        <p class="mb-0">@error</p>
    }
</alert>
```

Register in `_ViewImports.cshtml`:

```cshtml
@addTagHelper *, MyApp
```

> [!summary] Section Summary
> - Custom tag helpers can encapsulate complex Bootstrap patterns into clean, semantic HTML
> - `PreContent`, `PostContent`, and `Content` control where generated HTML is placed relative to child content
> - Properties like `Dismissible` and `Heading` provide a clean API surface
> - A single `@addTagHelper *, MyApp` registration enables all custom tag helpers in the assembly

---
