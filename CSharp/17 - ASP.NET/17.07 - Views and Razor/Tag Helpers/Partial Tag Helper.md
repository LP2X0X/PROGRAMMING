---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


The partial tag helper renders [[Partial Views and View Components|partial views]] inline:

```cshtml
<partial name="_ProductCard" model="product" />

@* With additional ViewData *@
<partial name="_ProductCard"
         model="product"
         view-data="@(new ViewDataDictionary(ViewData) { { "ShowPrice", true } })" />

@* Specifying the full path *@
<partial name="/Views/Shared/_Header.cshtml" />
```

> [!ad-note] Partial vs View Component Tag Helpers
> Do not confuse `<partial name="...">` (renders a partial view) with `<vc:component-name>` (invokes a view component). Partials receive their data from the calling view; view components fetch their own data.

> [!summary] Section Summary
> - `<partial name="..." model="..." />` renders a partial view inline
> - Optional `view-data` attribute passes additional `ViewDataDictionary` entries
> - Discovery follows the same rules as other partial rendering methods

---
