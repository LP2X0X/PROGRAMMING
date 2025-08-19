---
tags:
  - js
  - term
  - fundamental
---

In the **Web Audio API**, a **node** is a building block of the **[[Audio Processing Graph|audio processing graph]]**. Each node represents an audio processing module (like a source, effect, or destination), and nodes are connected together to route and transform audio.

---

## **🔹 Common Node Types**

|**Node Type**|**Description**|
|---|---|
|AudioBufferSourceNode|Plays an audio buffer (e.g., a loaded file)|
|OscillatorNode|Generates simple waveforms (sine, square, etc.)|
|GainNode|Controls volume (gain)|
|BiquadFilterNode|Filters audio (low-pass, high-pass, etc.)|
|AnalyserNode|Provides frequency/time-domain analysis for visualizations|
|PannerNode|Spatializes audio in 3D space|
|DelayNode|Delays audio for echo effects|
|DynamicsCompressorNode|Applies dynamic range compression|
|ScriptProcessorNode ⚠️|Old way to process audio with JavaScript (deprecated)|
|AudioWorkletNode|Modern custom audio processing in JavaScript|
|MediaElementAudioSourceNode|Connects an <audio> or <video> element to the graph|
|MediaStreamAudioSourceNode|Connects a live media stream (e.g., mic input)|

---

## **🔹 Basic Graph Example**

```js
const ctx = new AudioContext();

// Create nodes
const osc = ctx.createOscillator();     // Source node
const gain = ctx.createGain();          // Volume control
const analyser = ctx.createAnalyser();  // For visualization

// Connect nodes: osc -> gain -> analyser -> speakers
osc.connect(gain);
gain.connect(analyser);
analyser.connect(ctx.destination);

// Start sound
osc.start();
```

---

## **🔹 Anatomy of a Node**

Each node has:
- **Inputs/Outputs**: You connect nodes using .connect().
- **Parameters**: Many nodes expose AudioParams like gain.value or frequency.value.
- **Methods**: Some nodes (like OscillatorNode) have start() and stop().

---

## **🔹 Example: GainNode**

```js
const gainNode = ctx.createGain();
gainNode.gain.value = 0.5; // Set volume to 50%
```

---

## **🔸 How Nodes Connect**

You build an **audio graph** like this:

```ad-example
[OscillatorNode] → [GainNode] → [BiquadFilterNode] → [AudioDestinationNode]
```

Each node’s .connect() method links it to the next:

```js
osc.connect(gain).connect(filter).connect(ctx.destination);
```

---

## **🔸 Final Node:** 

## **AudioDestinationNode**

Every audio graph ends at the **audio output** of your device:

```js
ctx.destination
```
