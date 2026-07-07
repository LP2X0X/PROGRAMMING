---
tags:
  - uml
  - timing-diagram
  - behavioral
---

## 🔹 What It Shows

A timing diagram shows **state changes of objects or participants over time**. The horizontal axis represents time progressing left to right, and the vertical axis shows the discrete states or conditions each participant can be in. Transitions between states appear as vertical edges (instant) or sloped edges (gradual), and you can annotate exact durations and timing constraints.

Unlike [[Sequence Diagram]]s which emphasize message ordering, timing diagrams emphasize **when** things happen and **how long** they last.

> This is one of the **rarest UML diagrams** in everyday software work. It is most useful in **embedded systems, hardware design, and communication protocols** where precise timing matters. If you are doing web apps or business software, you will almost never draw one -- but when you need it, nothing else works as well.

## 🔹 When to Use

- **Real-time systems** -- tasks must complete within hard deadlines
- **Hardware/firmware design** -- clock signals, bus timing, chip-select sequences
- **Communication protocols** -- SPI, I2C, UART, TCP handshakes with timeout windows
- **Embedded systems** -- interrupt latency, sensor sampling windows
- **Signal timing analysis** -- verifying that setup/hold times are met

If your problem is "does participant B see the signal before deadline X?" then a timing diagram is the right tool.

## 🔹 Two Lifeline Formats

Timing diagrams support two visual formats for each lifeline. Both convey the same information; pick whichever reads more clearly for your audience.

### Value Lifeline Format (Concise)

A compact, single-line timeline. Each state is written as a label on a horizontal segment, with transition points marked.

```
         0s      5s     10s     15s     20s
Sensor:  |--Idle--|--Sampling--|--Idle--|--Sampling--|
```

Good for: quick sketches, few states, overview-level diagrams.

### State/Condition Timeline Format (Waveform)

A waveform-style chart where each state occupies a horizontal band and the lifeline steps between bands. This is the classic "digital logic analyzer" look.

```
          0s    5s    10s   15s   20s   25s   30s
          :     :     :     :     :     :     :
  High  --+     +-----+           +-----+
          |     |     |           |     |
  Low   --+-----+     +-----------+     +-----
```

Good for: detailed timing analysis, multiple interacting signals, hardware review.

## 🔹 Duration Constraints

Duration constraints specify **how long** a state must last. Notated with curly braces and the variable `d`:

```
{d = 5s}          -- exactly 5 seconds
{d >= 2s}         -- at least 2 seconds
{d <= 10ms}       -- at most 10 milliseconds
{5s <= d <= 10s}  -- between 5 and 10 seconds (the d..d' range)
```

Place the constraint label above or below the state segment it applies to.

```
         |<--- {d = 30s} --->|
  Red  --+-------------------+
         |                   |
  Green  +                   +------
```

## 🔹 Time Constraints

Time constraints pin an event to an **absolute point in time** (or a window). Notated with `t`:

```
{t = 0s}           -- event occurs at exactly t=0
{t <= 100ms}       -- event must occur by 100ms
{10ms <= t <= 20ms} -- event occurs between 10ms and 20ms (the t..t' range)
```

Use these to mark deadlines, setup/hold times, or protocol timeouts.

```
  Request ----+          {t <= 50ms}
              |               |
  Response    +- - - - - - - -+------
                  (must arrive by 50ms)
```

## 🔹 Messages Between Lifelines

Horizontal or angled arrows between lifelines represent messages or signals, just like in [[Sequence Diagram]]s but plotted against a time axis.

```
          0ms   10ms   20ms   30ms   40ms   50ms
          :      :      :      :      :      :
Client    :      :      :      :      :      :
  Idle  --+      +------+             +------+--
          |      |      |             |      |
  Wait  --+------+   +--+-------------+      :
               |     |        |
               |  SYN|        |
               +---->|        |
                     |SYN-ACK |
                     +------->|
                              | ACK
                              +-------->
Server    :      :      :      :      :      :
  Listen--+------+      :      :      :      :
          :      |      :      :      :      :
  SynRcvd :      +------+      :      :      :
          :      :      |      :      :      :
  Estab   :      :      +------+------+------+--
```

Arrow labels follow [[Common Notation]] conventions for messages.

## 🔹 Example: Traffic Light Timing Diagram

A single-lifeline diagram showing a traffic light cycle with duration constraints:

```
  Traffic Light Timing Diagram
  ============================

          0s        30s       35s       65s       70s
          :          :         :         :         :
          |<--{d=30s}->|<{d=5s}>|<-{d=30s}->|<{d=5s}>|
          :          :         :         :         :
  Red   --+----------+         :         +---------+--
          |          |         :         |         |
  Yellow  +          +----+    :    +----+         +--
          :               |   :    |
  Green   :               +---+----+
          :          :         :         :         :

  Cycle:  Red(30s) -> Green(30s) -> Yellow(5s) -> Red ...

  Notes:
  - Pedestrian signal ([[Interaction Overview Diagram]] for full flow)
  - Yellow duration constrained: {d = 5s}
  - Full cycle period: {d = 65s}
```

## 🔹 Example: SPI Protocol Handshake

Two lifelines (Master and Slave) with timing constraints:

```
  SPI Single-Byte Transfer
  =========================

        0       t1      t2      t3      t4
        :       :       :       :       :
  CS    :       :       :       :       :
  High -+       :       :       :       +---
        |       :       :       :       |
  Low   +-------+-------+-------+-------+
        : {t<=5ns}:     :       :       :
  CLK   :       :       :       :       :
  Low  -+--+  +-+--+  +-+--+  +-+--+  ++---
           |  |    |  |    |  |    |  |
  High     +--+    +--+    +--+    +--+
        :       :       :       :       :
  MOSI  : D7    : D6    : D5    : D4    :
  ------+/======+/======+/======+/======+---
        :       :       :       :       :
  MISO  : D7    : D6    : D5    : D4    :
  ------+/======+/======+/======+/======+---

  Constraints:
  - CS must go low {t >= 5ns} before first CLK edge
  - Data valid: {t <= 3ns} after CLK rising edge
  - Hold time: {d >= 2ns} after CLK falling edge
```

## 🔹 Quick Notation Reference

| Element                | Notation / Symbol         | Meaning                                         |
| ---------------------- | ------------------------- | ----------------------------------------------- |
| Lifeline               | Named row                 | Object or participant being tracked              |
| State / Condition      | Horizontal segment        | Participant is in this state                     |
| Transition             | Vertical / sloped edge    | State change (instant or gradual)                |
| Duration constraint    | `{d = 5s}`, `{d..d'}`    | How long a state lasts                           |
| Time constraint        | `{t = 0s}`, `{t..t'}`    | When an event must occur                         |
| Message                | Arrow between lifelines   | Signal or call from one participant to another   |
| Tick mark              | Short vertical mark       | Discrete time point on the axis                  |
| Time axis              | Horizontal, left to right | Progression of time                              |
| Destruction            | `X` on lifeline           | Participant ceases to exist                      |
| General value          | Label on segment          | Named state (e.g., `Idle`, `Active`, `High`)     |

## 🔹 Practical Tips

- **You will rarely draw these** unless you work in embedded, firmware, or protocol design. In a typical software career you might never need one.
- When you DO need one, a [[Sequence Diagram]] cannot replace it -- sequence diagrams show order but not duration or absolute time.
- Keep the number of lifelines small (2-4). More than that and the diagram becomes unreadable -- split into multiple diagrams.
- Combine with [[State Machine Diagram]]s: the timing diagram shows when transitions happen; the state machine shows the full transition logic.
- Use consistent time units across all lifelines (don't mix seconds and milliseconds without clear labels).
- Tools: most UML tools support timing diagrams poorly. Consider dedicated waveform tools (WaveDrom, Wavedrom-Editor) for hardware-level diagrams and PlantUML for simpler ones.
- In PlantUML, use `@startuml` with `robust` or `concise` keywords for the two lifeline formats.

See also: [[Common Notation]], [[Communication Diagram]], [[Sequence Diagram]], [[State Machine Diagram]], [[Interaction Overview Diagram]]
