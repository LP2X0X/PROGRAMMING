---
tags:
  - html
  - note
  - fundamental
---

- This is what we call "HTML boilerplate", which just means that no matter what type of webpage you are creating, it will ALWAYS follow this structure.

```html
<html>
  <head>
    <title>Site Title</title>
  </head>

  <body>
    <!-- This is an HTML comment 
    
    It looks different than our JavaScript comments
    and can be used across multiple lines

    -->
  </body>
</html>
```

- While this is all that is necessary for a webpage, here is a more "real world" version of HTML boilerplate.

```html
<!DOCTYPE html>

<html lang="en">
  <head>
    <meta charset="utf-8" />

    <title>Example Site</title>
    <meta name="description" content="An example HTML site" />
    <meta name="author" content="Zach Gollwitzer" />

    <link rel="stylesheet" href="css/styles.css" />
  </head>

  <body>
    <script src="js/scripts.js"></script>
  </body>
</html>
```
- Here are what the tags mean:
	- `!doctype html` - Tells the web browser to expect an HTML document. This is not an HTML element.
	- `lang="en"` - Tells search engines and web browsers what language this webpage is written in. [Here are all the valid country codes.](https://www.w3schools.com/tags/ref_country_codes.asp)
	- `charset="utf-8"` - This defines what character set to use and is a [very involved topic](https://www.w3schools.com/html/html_charset.asp).
	- `<html></html>` - The HTML container
	- `<head></head>` - Stores all the metadata about the webpage
	- `<title></title>` - Look up at the text in your browser's tabs. The contents of this will represent the tab name.
	- `<meta>` - Nope, not `<meta></meta>`. Some HTML tags are "self closing". This just defines useful information for the browser and search engines to use when viewing the webpage.
	- `<link>` - This allows you to connect your CSS stylesheet. We will be talking more about this in a future lesson.
	- `<script>` - This allows you to connect JavaScript code to your HTML document. We will cover this later in the post when we talk about the document object model (DOM).
	- \<body>\</body> - Contains all visible elements.