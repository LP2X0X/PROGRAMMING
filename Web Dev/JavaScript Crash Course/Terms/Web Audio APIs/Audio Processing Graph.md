---
tags:
  - js
  - term
  - fundamental
---

The **audio processing graph** in the **Web Audio API** is a **modular system** where individual audio units—called **AudioNodes**—are connected together to create complex audio processing workflows. Think of it like a **flowchart for sound**, where sound flows from sources, through processors, and finally to your speakers.

---

## **🎧 Audio Processing Graph = A Network of AudioNodes**

  

### **🔹 How It Works:**

- Each **AudioNode** performs a specific task (e.g., play a sound, change volume, apply a filter).
    
- You connect nodes using the .connect() method.
    
- The graph starts at a **source** and ends at the **destination** (your output device).
    

---

### **🔸 Visual Example**

```
[OscillatorNode] ─▶ [GainNode] ─▶ [BiquadFilterNode] ─▶ [AnalyserNode] ─▶ [AudioDestinationNode]
```

- **OscillatorNode**: generates sound (e.g., sine wave)
    
- **GainNode**: controls volume
    
- **BiquadFilterNode**: applies effects like low-pass filter
    
- **AnalyserNode**: used for visualizations
    
- **AudioDestinationNode**: sends the sound to your speakers
    

---

### **🔹 Code Example: Basic Audio Graph**

```js
const ctx = new AudioContext();

const osc = ctx.createOscillator();      // Sound source
const gain = ctx.createGain();           // Volume control
const filter = ctx.createBiquadFilter(); // Filter
const analyser = ctx.createAnalyser();   // For visualization

// Connect nodes to form a graph
osc.connect(gain);
gain.connect(filter);
filter.connect(analyser);
analyser.connect(ctx.destination);

// Start audio
osc.start();
```

---

### **🔹 Why It’s Powerful**

✅ Flexible: You can rearrange or add/remove nodes dynamically
✅ Real-time: All processing happens in real time in the browser
✅ Customizable: You can build effects, visualizations, synthesizers, etc.

---

### **🔹 Common Node Roles**

|**Node Type**|**Role**|
|---|---|
|OscillatorNode|Audio source|
|AudioBufferSourceNode|Play back recorded sound|
|GainNode|Volume control|
|BiquadFilterNode|Equalization/filtering|
|AnalyserNode|Visualization|
|PannerNode|Spatial audio|
|AudioDestinationNode|Output (usually speakers)|