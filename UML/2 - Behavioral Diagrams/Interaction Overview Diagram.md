---
tags:
  - uml
  - interaction-overview-diagram
  - behavioral
---

## 🔹 What It Shows

An Interaction Overview Diagram is a variant of an [[Activity Diagram]] where the nodes are **interaction fragments** -- small [[Sequence Diagram]]s or references to them. It orchestrates the control flow between multiple interaction diagrams, combining:

- The **control flow** mechanics of [[Activity Diagram]]s (decisions, forks, joins, merges)
- The **interaction modeling** of [[Sequence Diagram]]s and [[Communication Diagram]]s

Instead of activity nodes containing actions, each node contains (or references) an entire interaction scenario. The result is a high-level map showing how multiple interactions connect and flow.

## 🔹 When to Use

- **Orchestrating multiple interactions** -- you have several related [[Sequence Diagram]]s or [[Communication Diagram]]s and need to show how they connect
- **Complex multi-scenario workflows** -- a business process branches into different interaction scenarios depending on conditions
- **High-level system overviews** -- showing the overall flow of a system as a series of interaction steps without drowning in message-level detail

## 🔹 Rarity in Practice

This diagram type is **seldom used** in real projects. Most teams get by with [[Sequence Diagram]]s that use `ref` fragments to reference other interactions. The Interaction Overview Diagram exists in the UML spec, but the tooling support is limited and the added formalism rarely justifies the effort.

**Bottom line:** know it exists, understand the concept, but expect to use `ref` frames inside [[Sequence Diagram]]s instead.

## 🔹 Quick Reference: Elements

| Element | Symbol | Borrowed From | Purpose |
|---|---|---|---|
| Initial node | `●` (filled circle) | [[Activity Diagram]] | Entry point of the flow |
| Activity final node | `⊙` (bull's-eye) | [[Activity Diagram]] | End of the flow |
| Decision / Merge | Diamond `◇` | [[Activity Diagram]] | Branch or rejoin control flow |
| Fork / Join | Thick horizontal bar | [[Activity Diagram]] | Split into / synchronize parallel flows |
| Inline interaction | `sd` frame with messages | [[Sequence Diagram]] | Full interaction shown directly as a node |
| Referenced interaction | `ref` frame with name | [[Sequence Diagram]] | Pointer to a named interaction defined elsewhere |
| Flow edge | Arrow → | [[Activity Diagram]] | Connects nodes in sequence |
| Guard condition | `[condition]` on edge | Both | Controls which branch to take |

## 🔹 Inline vs Referenced Interactions

**Referenced interaction (`ref`)** -- a compact frame that points to a separately defined interaction diagram. Keeps the overview clean.

```
┌──────────────────────────┐
│ ref                      │
│                          │
│   Login Sequence         │
│                          │
└──────────────────────────┘
```

**Inline interaction (`sd`)** -- the full sequence diagram is drawn directly inside the node. Use sparingly or the overview becomes cluttered.

```
┌──────────────────────────┐
│ sd  PaymentFlow          │
│  :User      :PaySvc      │
│    │──pay()──>│          │
│    │<──ack────│          │
└──────────────────────────┘
```

In practice, **prefer `ref` frames** for the overview and define each interaction in its own [[Sequence Diagram]]. Inline interactions are only useful for very small, 2-3 message exchanges.

## 🔹 Real-World Example: E-Commerce Order Flow

```
                        ●
                        │
           ┌────────────┴────────────┐
           │ ref                     │
           │     Login Sequence      │
           └────────────┬────────────┘
                        │
                       ◇ [authenticated?]
                      / \
              [yes]  /   \  [no]
                    /     \
                   │       │
                   │    ┌──┴──────────────┐
                   │    │ ref             │
                   │    │  Registration   │
                   │    └──┬──────────────┘
                   │       │
                    \     /
                     \   /
                      ◇  (merge)
                      │
           ┌──────────┴──────────┐
           │ ref                 │
           │   Browse Catalog    │
           └──────────┬──────────┘
                      │
           ┌──────────┴──────────┐
           │ ref                 │
           │   Checkout Flow     │
           └──────────┬──────────┘
                      │
               ═══════╤═══════  (fork)
              │               │
   ┌──────────┴────┐  ┌──────┴──────────┐
   │ ref           │  │ ref             │
   │  Payment      │  │  Inventory      │
   │  Processing   │  │  Reservation    │
   └──────────┬────┘  └──────┬──────────┘
              │               │
               ═══════╤═══════  (join)
                      │
           ┌──────────┴──────────┐
           │ ref                 │
           │  Order Confirmation │
           └──────────┬──────────┘
                      │
                      ⊙
```

This shows how seven distinct interaction scenarios connect: Login, Registration, Browse, Checkout, Payment, Inventory, and Confirmation -- with a decision after login and parallel processing for payment and inventory.

## 🔹 When It Adds Value vs When It's Overkill

| Situation | Verdict |
|---|---|
| Complex system with 5+ distinct interaction scenarios that branch and merge | Adds value -- gives a map of how interactions relate |
| Simple linear flow (do A, then B, then C) | Overkill -- a single [[Sequence Diagram]] with `ref` frames is simpler |
| Small system with 2-3 interactions | Overkill -- just use `ref` fragments in one [[Sequence Diagram]] |
| Need to show parallel interaction paths with synchronization | Adds value -- fork/join semantics are explicit |
| Team already comfortable with [[Activity Diagram]]s and [[Sequence Diagram]]s | Worth considering -- the notation will be familiar |

**In practice:** most teams skip this diagram entirely and use `ref` fragments within [[Sequence Diagram]]s to achieve the same orchestration effect.

## 🔹 Tips

- **Use `ref` frames, not inline interactions** -- keep each node small and define the details in separate [[Sequence Diagram]]s
- **Think of it as a table of contents** -- the Interaction Overview Diagram is the index; the referenced interactions are the chapters
- **If you only have 2-3 interactions**, a single [[Sequence Diagram]] with `ref` frames is simpler and more widely understood
- **Guard conditions on edges** work exactly like in [[Activity Diagram]]s -- use `[condition]` notation
- **Fork/join bars** indicate true parallelism, not just "do these in any order"

---

See also: [[Activity Diagram]], [[Sequence Diagram]], [[Communication Diagram]]
