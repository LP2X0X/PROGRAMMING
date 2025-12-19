---
tags: 
 - html
 - element
---

### HTML `<article>` — compact overview

#### **What it is**  
The \<article> HTML element represents a self-contained composition in a document, page, application, or site, which is intended to be independently distributable or reusable (e.g., in syndication). Examples include: a forum post, a magazine or newspaper article, or a blog entry, a product card, a user-submitted comment, an interactive widget or gadget, or any other independent item of content.

---

#### **When to use it**

- Use for content that makes sense on its own outside the page (RSS, bookmarks, syndication).
    
- Don’t use it just for styling — use it for meaningful, standalone pieces of content.
    

---

#### **Typical contents**

- Heading, author, timestamp, paragraphs, images, media, related links.
    
- Can contain other sectioning elements (`<section>`, `<aside>`, `<header>`, `<footer>`).
    

---

#### **Semantic differences**

- `<article>` = an independent, self-contained item.
    
- `<section>` = a thematic grouping within a document (not necessarily independently distributable).
    
- `<aside>` = tangential or complementary content (sidebars, pull quotes).
    
- Multiple `<article>` elements can appear on one page (e.g., list of posts).
    

---

#### **Best practices**

- Give each article a clear heading.
    
- If article can be embedded/shared, include enough context (title, author, date).
    
- Don’t nest `<article>` inside `<article>` unless it truly is an independent sub-article (e.g., an article that contains embedded standalone posts).
    
- Use `<article>` inside lists when each list item is a full post (e.g., newsfeed).
    

**Small example**

```html
<article id="post-123" class="post">
  <header>
    <h2>How to Brew Great Coffee at Home</h2>
    <p>By <a href="/author/jane">Jane</a> — <time datetime="2025-11-30">Nov 30, 2025</time></p>
  </header>

  <p>Intro paragraph...</p>
  <section>
    <h3>What you need</h3>
    <ul><li>Grinder</li><li>Coffee beans</li></ul>
  </section>

  <footer>
    <p>Tags: <a href="/tag/coffee">coffee</a></p>
  </footer>
</article>
```