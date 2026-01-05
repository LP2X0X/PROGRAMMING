---
tags: 
 - html
 - attribute
---

The `muted` attribute is a Boolean attribute that specifies that the audio output should be silenced by default when the page loads.

---

## 1. Usage and Syntax

When the `muted` attribute is present, the audio will play (the timeline will progress), but the user will hear nothing until they manually "unmute" it via the controls or your custom interface.

```html
<audio src="background-track.mp3" controls muted>
  Your browser does not support the audio element.
</audio>
```

---

## 2. Why use the Muted attribute?

### A. The "Autoplay" Key

As discussed previously, the primary technical use for `muted` is to bypass browser autoplay restrictions.

- **The Rule:** Browsers generally block audio from playing automatically to prevent a "loud surprise" for the user.
    
- **The Loophole:** If the audio is `muted`, browsers will almost always allow it to `autoplay` immediately.
    

### B. User Experience (UX)

Muting by default is a "user-first" design choice. It allows users to decide if and when they want to consume audio content, which is especially important for:

- Users in public spaces (offices, libraries).
    
- Users using screen readers (who don't want audio overlapping with their accessibility software).
    
- Background ambient sounds that might be distracting if played at full volume immediately.
    

---

## 3. Interaction with JavaScript

You can programmatically check or change the muted state using the `.muted` property.

```js
const myAudio = document.querySelector('audio');

// Mute the audio via script
myAudio.muted = true;

// Toggle mute/unmute
function toggleMute() {
  myAudio.muted = !myAudio.muted;
}
```

> **Note:** The `muted` attribute only sets the _initial_ state. If a user unmutes the audio and then refreshes the page, the audio will return to being muted (unless you use `localStorage` to save their preference).

---

## 4. Visual Representation

When `muted` is active and the `controls` attribute is present, the browser will display a "slashed" speaker icon.

---

## 5. Comparison: `muted` vs. `volume="0"`

| **Feature**  | **muted attribute**                             | **volume property**                             |
| ------------ | ----------------------------------------------- | ----------------------------------------------- |
| **Type**     | Boolean (on/off)                                | Numeric ($0.0$ to $1.0$)                        |
| **Function** | Silences audio without changing volume level.   | Sets the actual loudness.                       |
| **Autoplay** | Required for autoplay to work in most browsers. | Usually does not satisfy autoplay bypass rules. |