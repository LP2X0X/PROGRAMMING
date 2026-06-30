---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


These tag helpers add **cache-busting** via file version hashing.

### Image Tag Helper

```cshtml
<img src="~/images/logo.png" asp-append-version="true" />
```

Renders:

```html
<img src="/images/logo.png?v=Kl_dqr9NTwpYS27JkWm3Eg" />
```

The `v` query parameter is a hash of the file contents. When the file changes, the hash changes, forcing browsers to download the updated file instead of serving a stale cached version.

### Link and Script Tag Helpers

```cshtml
<link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
<script src="~/js/site.js" asp-append-version="true"></script>
```

### Script Tag Helper with Fallback

```cshtml
<script src="https://cdn.example.com/lib/jquery.min.js"
        asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
        asp-fallback-test="window.jQuery">
</script>
```

If the CDN fails, the fallback local script loads automatically.

> [!tip] Practical Tip
> Always use `asp-append-version="true"` on your CSS and JS references in production layouts. It eliminates the most common class of "I deployed but users see the old version" bugs with zero effort.

> [!summary] Section Summary
> - `asp-append-version="true"` adds a content-based hash query parameter for cache-busting
> - Works on `<img>`, `<link>`, and `<script>` elements
> - The hash changes when the file changes, forcing browser cache invalidation
> - `asp-fallback-src` and `asp-fallback-test` provide CDN fallback behavior for scripts

---
