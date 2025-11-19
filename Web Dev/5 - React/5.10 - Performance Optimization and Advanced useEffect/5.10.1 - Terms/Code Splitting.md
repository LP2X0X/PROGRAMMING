---
tags: 
 - react
 - term
---

- **Code splitting** is the process of **dividing a large bundle into smaller chunks** so the app only loads the code it needs, improving performance and load time.

![[Pasted image 20251110211145.png|center|700]]

- Code-splitting your app can help you “lazy-load” just the things that are currently needed by the user, which can dramatically improve the performance of your app. While you haven’t reduced the overall amount of code in your app, you’ve avoided loading code that the user may never need, and reduced the amount of code needed during the initial load.