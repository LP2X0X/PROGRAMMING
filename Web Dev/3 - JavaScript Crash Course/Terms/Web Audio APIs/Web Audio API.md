---
tags: js, term, fundamental
---

The **Web Audio API** is a powerful JavaScript API for creating, processing, and controlling audio directly in the browser. It’s widely used in games, music apps, and sound visualizations.

---

## **🔹 Core Concepts**

### **1. AudioContext**

The main controller that manages and connects audio operations.

```
const audioCtx = new AudioContext();
```

---

### **2. Nodes**

Everything in Web Audio is built with **nodes** connected together in a graph. Common node types:

|**Node Type**|**Purpose**|
|---|---|
|AudioBufferSourceNode|Play audio files|
|OscillatorNode|Generate tones (sine, square, etc.)|
|GainNode|Control volume|
|BiquadFilterNode|Apply filters (low-pass, etc.)|
|AnalyserNode|For visualizations (FFT data)|

---

## **🔹 Example: Play a Tone**

```js
const audioCtx = new AudioContext();

const oscillator = audioCtx.createOscillator(); // Create a tone generator
oscillator.type = 'sine'; // waveform type: sine, square, triangle, sawtooth
oscillator.frequency.setValueAtTime(440, audioCtx.currentTime); // A4 note

oscillator.connect(audioCtx.destination); // Connect to speakers
oscillator.start();
oscillator.stop(audioCtx.currentTime + 1); // Play for 1 second
```

---

## **🔹 Example: Load and Play Audio File**

```js
async function playSound(url) {
  const audioCtx = new AudioContext();
  const response = await fetch(url);
  const arrayBuffer = await response.arrayBuffer();
  const audioBuffer = await audioCtx.decodeAudioData(arrayBuffer);

  const source = audioCtx.createBufferSource();
  source.buffer = audioBuffer;
  source.connect(audioCtx.destination);
  source.start();
}

playSound('sound.mp3');
```

---

## **🔹 Volume Control with GainNode**

```js
const gainNode = audioCtx.createGain();
gainNode.gain.value = 0.5; // 50% volume

source.connect(gainNode);
gainNode.connect(audioCtx.destination);
```

---

## **🔹 Visualization with AnalyserNode**

You can analyze and visualize audio in real-time using AnalyserNode and Canvas.

---

## **🔸 Notes**

- Browsers **require user interaction** to start AudioContext (e.g., a click).
- You can chain multiple nodes: source → gain → filter → analyser → destination.
