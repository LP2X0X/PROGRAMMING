---
tags: 
 - html
 - attribute
---

##  `preload`

The `preload` attribute provides a "hint" to the browser about how much of the file should be downloaded when the page loads. This is a crucial tool for **page speed optimization**.

- **`none`**: The browser does not load the audio/video at all until the user clicks "Play." This saves the most bandwidth.
    
- **`metadata`**: The browser only downloads basic info (duration, file size, dimensions). This is the **best practice** for most websites because it shows the user how long the track is without a heavy download.
    
- **`auto`**: The browser downloads the entire file immediately. Use this only if the media is the primary purpose of the page (e.g., a specific landing page for a podcast episode).
    
