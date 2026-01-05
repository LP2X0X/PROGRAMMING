---
tags: 
  - html
  - attribute
---

The `controls` attribute is a Boolean attribute that tells the browser to display its built-in playback interface. Without this attribute, the audio element is invisible to the user unless you create custom buttons using JavaScript.

### What it Includes

When you add the `controls` keyword to your `<audio>` tag, the browser typically renders:

- **Play/Pause button:** To start and stop the audio.
    
- **Seek bar (Timeline):** To jump to specific parts of the track.
    
- **Volume slider:** To adjust or mute the sound.
    
- **Time display:** Showing current playback time and total duration.
    
- **Playback speed:** (In most modern browsers) Options to speed up or slow down the audio.
    

### Usage Syntax

You don't need to give it a value; simply adding the word is enough:

```html
<audio src="music.mp3" controls></audio>
```

### Important Considerations

- **Browser Consistency:** Every browser (Chrome, Safari, Firefox) designs its own unique set of controls. They will look slightly different depending on the user's device and OS.
    
- **Layout Impact:** Adding the `controls` attribute gives the element a default width and height. If you remove it, the element collapses to $0 \times 0$ pixels.
    
- **Customization:** You cannot style the individual buttons (like the play button) inside the default browser controls using CSS. If you need a specific look, you must omit the `controls` attribute and build your own UI with the JavaScript Media API.
    