---
tags: 
 - html
 - attribute
---

The `autoplay` attribute is a Boolean attribute that instructs the browser to start playing the audio as soon as it has loaded enough data to do so without buffering.

While it sounds straightforward, modern web browsers have strict **Autoplay Policies** to prevent websites from annoying users with unexpected sound.

---

## 1. How to Use It

Like `controls`, you simply add the attribute to the opening tag:

```html
<audio src="notification.mp3" autoplay></audio>
```

---

## 2. The "Muted" Requirement

Most modern browsers (Chrome, Edge, Safari) will **block** autoplay if the audio has a sound track and the user hasn't interacted with the page yet (like clicking or tapping).

To guarantee that an audio file autoplays, it usually must be combined with the `muted` attribute:

```html
<audio src="ambient-background.mp3" autoplay muted></audio>
```

_Note: This is more common for video. For audio, since muting it defeats the purpose of "listening," autoplay is rarely the best choice for primary content._

---

## 3. Best Practices & UX

- **User Consent:** Never autoplay music on a content-heavy page (like a blog post). It’s considered a major accessibility and UX failure.
    
- **Engagement:** Autoplay is generally acceptable in "app-like" scenarios. For example, if a user clicks "Start Game," the next page can autoplay the background music because the user has already expressed intent.
    
- **Battery & Data:** On mobile devices, autoplaying files can consume a user's data plan and battery life without their explicit permission.
    

---

## 4. How Browsers Decide (The MEI)

Browsers use something called a **Media Engagement Index (MEI)**. This is a local score the browser keeps for every website you visit. If you frequently play audio/video on a specific site (like YouTube or Spotify), the browser will eventually "trust" that site and allow it to autoplay audio without being muted.

---

## 5. JavaScript "Workaround"

If you try to play audio via JavaScript immediately on page load, you will likely see this error in your console:

Uncaught (in promise) DOMException: play() failed because the user didn't interact with the document first.

**The Solution:** Trigger the audio inside a user event, like a "Enter Site" button:

```js
const btn = document.querySelector('#start-btn');
const audio = document.querySelector('audio');

btn.addEventListener('click', () => {
  audio.play(); // This will work because it's inside a click event
});
```
