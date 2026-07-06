---
tags:
 - csharp
 - collections
 - generics
---

## What Is a Queue?

A `Queue<T>` is a collection that maintains items in a **FIFO (First-In, First-Out)** order. The first item you put in is the first item you take out — like a line at a store.

```
Enqueue("A") → Enqueue("B") → Enqueue("C")

  front                    back
  ┌───┬───┬───┐
  │ A │ B │ C │
  └───┴───┴───┘
  ↑               ↑
  Dequeue next    Enqueue here

Dequeue() → returns "A"
Dequeue() → returns "B"
Dequeue() → returns "C"
```

![[Pasted image 20240617093629.png|center|700]]

---

## Key Methods

| Method | Description |
|---|---|
| `Enqueue(T item)` | Add item to the back of the queue |
| `Dequeue()` | Remove and return the front item. Throws `InvalidOperationException` if empty |
| `Peek()` | Return the front item without removing it. Throws if empty |
| `TryDequeue(out T result)` | Try to dequeue — returns `false` instead of throwing if empty |
| `TryPeek(out T result)` | Try to peek — returns `false` instead of throwing if empty |
| `Count` | Number of items currently in the queue |
| `Contains(T item)` | Check if an item exists in the queue |
| `Clear()` | Remove all items |
| `ToArray()` | Copy queue to an array (front of queue = index 0) |

---

## Basic Usage

```csharp
Queue<string> queue = new Queue<string>();

queue.Enqueue("Alice");
queue.Enqueue("Bob");
queue.Enqueue("Charlie");

Console.WriteLine(queue.Peek());     // "Alice" — front of the line
Console.WriteLine(queue.Dequeue());  // "Alice" — removed
Console.WriteLine(queue.Dequeue());  // "Bob"
Console.WriteLine(queue.Count);      // 1 — only "Charlie" left

// Safe access — no exception if empty
if (queue.TryDequeue(out string next))
    Console.WriteLine(next);          // "Charlie"
if (queue.TryDequeue(out string val))
    Console.WriteLine(val);           // doesn't execute — queue is empty
```

---

## Processing All Items

A common pattern — process items until the queue is empty:

```csharp
Queue<string> orders = new Queue<string>();
orders.Enqueue("Order A");
orders.Enqueue("Order B");
orders.Enqueue("Order C");

while (orders.Count > 0)
{
    string order = orders.Dequeue();
    Console.WriteLine($"Processing: {order}");
}
// Processing: Order A
// Processing: Order B
// Processing: Order C
```

---

## Iterating a Queue

You can `foreach` over a queue (front to back), but you **cannot** access by index:

```csharp
Queue<int> queue = new Queue<int>();
queue.Enqueue(10);
queue.Enqueue(20);
queue.Enqueue(30);

foreach (int item in queue)
    Console.Write(item + " ");  // 10 20 30 — front to back

// queue[0]  ← compile error — no indexer on Queue<T>
```

> [!note]
> `foreach` does **not** remove items. It just reads through the queue. Only `Dequeue()` removes items.

---

## Queue vs Stack

| | Queue<T> | Stack<T> |
|---|---|---|
| Order | FIFO — first in, first out | LIFO — last in, first out |
| Add | `Enqueue()` — back | `Push()` — top |
| Remove | `Dequeue()` — front | `Pop()` — top |
| Analogy | Line at a store | Stack of plates |

```
Queue:  A → B → C     Dequeue → A (first in)
Stack:  A → B → C     Pop → C (last in)
```

---

## Common Use Cases

- **Task/job scheduling** — process tasks in the order they arrive
- **BFS (Breadth-First Search)** — traverse graphs/trees level by level
- **Message queues** — handle messages in arrival order
- **Print queue** — print documents in the order submitted
- **Rate limiting / buffering** — hold requests and process them in order

---

## Time Complexity

| Operation | Time |
|---|---|
| Enqueue | O(1) amortized |
| Dequeue | O(1) |
| Peek | O(1) |
| Contains | O(n) |

---

## Internal Implementation

Like `List<T>`, a queue uses an internal array. But instead of always adding/removing from one end, it uses a **circular buffer** with two pointers:

```
// After Enqueue A, B, C, D:
array: [ A | B | C | D |   |   |   |   ]
         ↑               ↑
        head            tail

// After Dequeue A, B:
array: [   |   | C | D |   |   |   |   ]
               ↑       ↑
              head    tail

// head moves forward instead of shifting all elements
// When tail reaches the end, it wraps around to the front
```

This is why `Enqueue` and `Dequeue` are both **O(1)** — no elements need to be shifted.