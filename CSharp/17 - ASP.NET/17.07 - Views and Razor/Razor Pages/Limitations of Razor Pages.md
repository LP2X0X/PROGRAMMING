---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


1. **Complex pages get messy**: a page with many handlers and properties can become a "god class"
2. **Testing isolation**: while PageModel is testable, the tight coupling of handlers to page rendering makes some tests less clean than controller tests
3. **Not great for APIs**: Razor Pages are designed for HTML rendering, not JSON APIs (use controllers or Minimal APIs)
4. **No multi-view support**: each page renders exactly one view, unlike a controller that can return different views from different actions
5. **Large teams**: the file-per-page model can lead to very large `/Pages/` directories in big applications
6. **Complex workflows**: multi-step wizards or approval flows with branching logic are easier to model with controllers

> [!warning] Common Misconception
> Razor Pages is not a "simplified" or "lesser" version of MVC. It is a **different programming model** for a different class of problems. Choosing Razor Pages for a CRUD admin panel is not "dumbing down" the architecture -- it is using the right tool for the job.

> [!summary] Section Summary
> - Razor Pages can become unwieldy for pages with many handlers ("god PageModel")
> - Not suitable for APIs -- use controllers or Minimal APIs
> - Single view per page (no multi-view support like controllers)
> - Complex multi-step workflows are better served by MVC controllers
> - Razor Pages is a valid architectural choice, not a simplified alternative

---
