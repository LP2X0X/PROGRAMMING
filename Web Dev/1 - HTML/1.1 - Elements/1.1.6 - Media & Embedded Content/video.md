---
tags: 
 - html
 - element 
 - media
---

The `<video>` element is the HTML5 standard for embedding video content natively in a web page. Much like the `<audio>` element, it removed the need for third-party plugins (like Flash) and provides a powerful API for developers to control playback.

---

## 1. Core Syntax & Multiple Sources

While a simple `src` attribute works, the best practice is to use nested `<source>` tags. This allows you to provide different file formats so the browser can choose the one it supports best.

```html
<video width="640" height="360" controls poster="preview-image.jpg">
  <source src="movie.webm" type="video/webm">
  <source src="movie.mp4" type="video/mp4">
  <track src="captions_en.vtt" kind="captions" srclang="en" label="English">
  Your browser does not support the video tag.
</video>
```

### Key Components:

- **`width` & `height`**: Sets the display size. Specifying these helps the browser reserve space on the page before the video loads, preventing "layout shift."
    
- **`<source>`**: Lists different versions of the video.1 The browser reads from top to bottom and plays the first compatible format.
    
- **`<track>`**: Used for subtitles, captions, or descriptions (crucial for accessibility).
    
- **Fallback Text**: The text inside the tags only appears if the user's browser is outdated.
    

---

## 2. Essential Video Attributes

Beyond the basics shared with audio, the video element has specific behaviors:

| **Attribute**      | **Behavior**                                                          | **Best Practice**                                   |
| ------------------ | --------------------------------------------------------------------- | --------------------------------------------------- |
| **`playsinline`**  | Plays video inside the page on mobile instead of forced full-screen.  | Essential for background videos or small UI clips.  |
| **`controlslist`** | Can restrict specific UI buttons like `nodownload` or `nofullscreen`. | Use sparingly; users generally prefer full control. |
| **`muted`**        | Required by most browsers (Chrome, Safari) for `autoplay` to work.    | Use for decorative background videos.               |
| **`object-fit`**   | (CSS property) Controls how the video scales into its container.      | Use `cover` for full-screen backgrounds.            |

---

## 3. Best Practices 

### A. Prioritize Efficiency with WebM and AV1

While **MP4 (H.264)** is the most compatible, **WebM (VP9)** or **AV1** offer much better compression. Smaller files mean faster load times and less data usage for your visitors. Always list your WebM/AV1 source _above_ the MP4 source.

### B. Accessibility is Non-Negotiable

Provide captions via `.vtt` files. This isn't just for users with hearing impairments; many users watch videos on "mute" in public spaces or while multitasking.

### C. Performance & Preloading

Don't use `preload="auto"` unless the video is the main content of the page. For background videos, use `preload="none"` or `preload="metadata"` to keep your initial page load light.

### D. Responsive Sizing

Instead of hardcoding pixels in HTML, use CSS to make your video responsive so it looks good on both phones and desktops:

```css
video {
  max-width: 100%;
  height: auto;
}
```

---

## 4. Video Formats Support (2026)

| **Format** | **Extension** | **Notes**                                               |
| ---------- | ------------- | ------------------------------------------------------- |
| **MP4**    | `.mp4`        | Universal support; best "catch-all" fallback.           | 
| **WebM**   | `.webm`       | Smaller files; supported by Chrome, Firefox, and Edge.  |
| **Ogg**    | `.ogv`        | Open-source, but becoming less common in favor of WebM. |

---

## 5. Responsive HTML5 video and iframes 

The only problem with our lovely HTML video implementation is that it’s not responsive. Thankfully, for HTML embedded video, the fix is easy. 

```css
video {
    max-width: 100%;
    height: auto;
}
```