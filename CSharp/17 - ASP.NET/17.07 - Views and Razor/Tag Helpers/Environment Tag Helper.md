---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


The **environment tag helper** conditionally renders content based on the current hosting environment:

```cshtml
<environment include="Development">
    <link rel="stylesheet" href="~/css/site.css" />
    <script src="~/lib/jquery/dist/jquery.js"></script>
</environment>

<environment exclude="Development">
    <link rel="stylesheet"
          href="https://cdn.example.com/css/site.min.css"
          asp-fallback-href="~/css/site.min.css"
          asp-fallback-test-class="sr-only"
          asp-fallback-test-property="position"
          asp-fallback-test-value="absolute" />
    <script src="https://cdn.example.com/lib/jquery.min.js"
            asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
            asp-fallback-test="window.jQuery">
    </script>
</environment>

<environment include="Staging,Production">
    <script>
        // Analytics tracking code
    </script>
</environment>
```

The `include` attribute accepts a comma-separated list of environment names. The content is only rendered when the current `ASPNETCORE_ENVIRONMENT` matches.

> [!summary] Section Summary
> - `<environment include="Development">` renders content only in specified environments
> - `<environment exclude="Production">` renders content in all environments except the specified ones
> - Common use: unminified assets in Development, CDN + minified assets in Production
> - Environment is determined by the `ASPNETCORE_ENVIRONMENT` variable

---
