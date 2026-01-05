---
tags: 
 - web dev
---

##  `loop`

The `loop` attribute is a Boolean that tells the browser to start the media over automatically as soon as it reaches the end.

- **Common Use Cases:** Background ambient noise, lo-fi music loops, or decorative background videos (cinemagraphs).
    
- **Behavior:** It creates a seamless transition back to the start. However, if your audio file has a split-second of silence at the end, the "loop" will feel staggered.
    
- **Code Example:**
    
    ```html
    <audio src="rain-sounds.mp3" autoplay loop muted></audio>
    ```
    
