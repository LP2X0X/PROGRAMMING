---
tags: 
 - html
 - attribute
---

##  `poster` (Video Only)

The `poster` attribute specifies an image to be shown while the video is downloading, or until the user hits the play button.

- **Why it's important:** If you don't provide a poster, the browser might just show a black box or the very first frame of the video, which might be blurry or unappealing.
    
- **Best Practice:** Use a high-quality `.jpg` or `.webp` that acts as a "thumbnail" for the video.
    
- **Syntax:**
    
    ```html
    <video controls poster="thumbnail.jpg">
      <source src="movie.mp4" type="video/mp4">
    </video>
    ```
    

---

### A Note on the "Poster" for Audio

While the HTML5 `<audio>` tag does not support the `poster` attribute, you can mimic this effect by placing an `<img>` tag above your audio player or using a `<div>` with a background image to create a "Now Playing" album art feel.