---
tags:
 - csharp
 - collections
 - generics
---

- In many ways, working with ObservableCollection\<T> is identical to working with List\<T>, given that both of these classes implement the same core interfaces. What makes the ObservableCollection\<T> class unique is that this class supports an event named CollectionChanged. **This event will fire whenever a new item is inserted, a current item is removed (or relocated), or the entire collection is modified.**
- Lives in `System.Collections.ObjectModel` namespace.

---

## Basic Usage

```csharp
using System.Collections.ObjectModel;
using System.Collections.Specialized;

var people = new ObservableCollection<string>();

people.CollectionChanged += (sender, e) =>
{
    Console.WriteLine($"Action: {e.Action}");
};

people.Add("Long");       // Action: Add
people.Add("Huy");        // Action: Add
people.Remove("Long");    // Action: Remove
people.Clear();           // Action: Reset
```

---

## CollectionChanged Event

The event provides `NotifyCollectionChangedEventArgs` with these properties:

| Property | Description |
|---|---|
| `Action` | What happened — `Add`, `Remove`, `Replace`, `Move`, `Reset` |
| `NewItems` | Items that were added (null for Remove/Reset) |
| `OldItems` | Items that were removed (null for Add/Reset) |
| `NewStartingIndex` | Index where new items were inserted |
| `OldStartingIndex` | Index where old items were before removal/move |

```csharp
people.CollectionChanged += (sender, e) =>
{
    switch (e.Action)
    {
        case NotifyCollectionChangedAction.Add:
            Console.WriteLine($"Added: {e.NewItems[0]} at index {e.NewStartingIndex}");
            break;
        case NotifyCollectionChangedAction.Remove:
            Console.WriteLine($"Removed: {e.OldItems[0]} from index {e.OldStartingIndex}");
            break;
        case NotifyCollectionChangedAction.Replace:
            Console.WriteLine($"Replaced: {e.OldItems[0]} → {e.NewItems[0]}");
            break;
        case NotifyCollectionChangedAction.Move:
            Console.WriteLine($"Moved: {e.OldStartingIndex} → {e.NewStartingIndex}");
            break;
        case NotifyCollectionChangedAction.Reset:
            Console.WriteLine("Collection was cleared");
            break;
    }
};
```

---

## INotifyPropertyChanged — The Other Event

`ObservableCollection<T>` also implements `INotifyPropertyChanged`, which fires when the `Count` or `Item[]` (indexer) properties change:

```csharp
((INotifyPropertyChanged)people).PropertyChanged += (sender, e) =>
{
    Console.WriteLine($"Property changed: {e.PropertyName}");
};

people.Add("Long");
// Output:
//   Property changed: Count
//   Property changed: Item[]
```

---

## Modifying Items Inside the Collection Does NOT Fire Events

This is a common gotcha. The collection only tracks **structural changes** (add, remove, move, replace). Changing a property on an item already in the collection fires **nothing**:

```csharp
public class Person
{
    public string Name { get; set; }
}

var people = new ObservableCollection<Person>();
var p = new Person { Name = "Long" };
people.Add(p);          // fires CollectionChanged

p.Name = "Huy";         // nothing — the collection doesn't know
```

To detect property changes on items, each item must implement `INotifyPropertyChanged` itself — and you must subscribe to each item's `PropertyChanged` event manually (or use a framework that does this for you).

---

## ObservableCollection\<T> vs List\<T>

| | `List<T>` | `ObservableCollection<T>` |
|---|---|---|
| Core behavior | Dynamic array | Dynamic array + change notifications |
| `CollectionChanged` event | No | Yes |
| `INotifyPropertyChanged` | No | Yes |
| Performance | Faster — no event overhead | Slightly slower — fires events on every mutation |
| Common use | General-purpose | UI data binding (WPF, MAUI, WinUI) |
| Bulk operations (`AddRange`) | Yes | No — must add one at a time |

---

## No AddRange

`ObservableCollection<T>` has no `AddRange`. Every `Add` fires a separate `CollectionChanged` event. Adding 1000 items fires 1000 events.

Workaround — subclass and override:

```csharp
public class RangeObservableCollection<T> : ObservableCollection<T>
{
    public void AddRange(IEnumerable<T> items)
    {
        foreach (var item in items)
            Items.Add(item);   // Items is the internal List<T>, no event per item

        OnCollectionChanged(
            new NotifyCollectionChangedEventArgs(NotifyCollectionChangedAction.Reset));
    }
}
```

`Items` is a protected `List<T>` inherited from `Collection<T>`. Writing to it directly bypasses the event. You then fire one `Reset` event at the end to tell the UI to refresh.

---

## When to Use

- **WPF / MAUI / WinUI data binding** — the UI framework listens to `CollectionChanged` to automatically update the view when the collection changes. This is the primary use case.
- **Any scenario where other code needs to react to collection mutations** — the observer pattern built in.

```ad-tip
If you don't need change notifications, use `List<T>`. `ObservableCollection<T>` adds overhead that only pays off when something is listening.
```