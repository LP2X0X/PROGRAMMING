---
tags:
 - csharp
 - collections
 - generics
---

## What Is This Namespace For?

`System.Collections.ObjectModel` provides **wrapper collections** that let you control or restrict how consumers interact with your data. The most important type here is `ObservableCollection<T>`, but the namespace also provides base classes for building your own custom collections.

Think of it this way:
- `System.Collections.Generic` — the collections you **use** internally
- `System.Collections.ObjectModel` — the collections you **expose** to others, with controlled access

---

## Key Types

| Type | Description |
|---|---|
| `ObservableCollection<T>` | A `List<T>`-like collection that **fires events** when items are added, removed, or the whole list is refreshed. Essential for data binding in WPF/MAUI/Blazor. |
| `ReadOnlyCollection<T>` | A read-only wrapper around an existing `IList<T>`. Consumers can read but not modify. |
| `ReadOnlyObservableCollection<T>` | Read-only wrapper around an `ObservableCollection<T>`. Still fires change notifications but prevents modification. |
| `Collection<T>` | A base class you can inherit from to build custom collections with override points (`InsertItem`, `RemoveItem`, `SetItem`, `ClearItems`). |
| `KeyedCollection<TKey, TItem>` | A collection where each item has an embedded key (extracted by overriding `GetKeyForItem`). Combines list-style indexed access with dictionary-style key lookup. |

---

## `ObservableCollection<T>` — The Most Used Type

Behaves like `List<T>` but raises `CollectionChanged` and `PropertyChanged` events whenever the collection is modified. This is what makes UI data binding work — the UI automatically updates when items change.

```csharp
using System.Collections.ObjectModel;
using System.Collections.Specialized;

ObservableCollection<string> names = new ObservableCollection<string>();

// Subscribe to changes
names.CollectionChanged += (sender, e) =>
{
    Console.WriteLine($"Action: {e.Action}");
    if (e.NewItems != null)
        foreach (var item in e.NewItems)
            Console.WriteLine($"  Added: {item}");
    if (e.OldItems != null)
        foreach (var item in e.OldItems)
            Console.WriteLine($"  Removed: {item}");
};

names.Add("Alice");     // Action: Add, Added: Alice
names.Add("Bob");       // Action: Add, Added: Bob
names.Remove("Alice");  // Action: Remove, Removed: Alice
names[0] = "Charlie";   // Action: Replace, Removed: Bob, Added: Charlie
```

### `NotifyCollectionChangedAction` values

| Action | When |
|---|---|
| `Add` | Item(s) added |
| `Remove` | Item(s) removed |
| `Replace` | Item replaced via indexer |
| `Move` | Item moved to a different index |
| `Reset` | Collection was cleared or drastically changed |

---

## `ReadOnlyCollection<T>` — Expose Without Allowing Modification

Wrap an internal list so callers can read it but not change it:

```csharp
List<string> internalList = new List<string> { "Alice", "Bob", "Charlie" };
ReadOnlyCollection<string> readOnly = new ReadOnlyCollection<string>(internalList);

Console.WriteLine(readOnly[0]);    // "Alice" — reading works
Console.WriteLine(readOnly.Count); // 3

// readOnly.Add("Diana");   // compile error — no Add method
// readOnly[0] = "Eve";     // compile error — no setter

// But the internal list still works — changes are reflected
internalList.Add("Diana");
Console.WriteLine(readOnly.Count); // 4 — it's a wrapper, not a copy
```

> [!warning]
> `ReadOnlyCollection<T>` is a **wrapper**, not a snapshot. If the underlying list changes, the read-only view reflects those changes. It only prevents consumers from modifying through the wrapper.

---

## `Collection<T>` — Base Class for Custom Collections

Inherit from `Collection<T>` when you want a `List<T>`-like collection but need to add validation or custom behavior on insert/remove:

```csharp
public class UniqueCollection<T> : Collection<T>
{
    protected override void InsertItem(int index, T item)
    {
        if (Contains(item))
            throw new ArgumentException("Duplicate item");
        base.InsertItem(index, item);
    }

    protected override void SetItem(int index, T item)
    {
        if (Contains(item))
            throw new ArgumentException("Duplicate item");
        base.SetItem(index, item);
    }
}

var col = new UniqueCollection<string>();
col.Add("Alice");
col.Add("Bob");
// col.Add("Alice");  // throws ArgumentException
```

### Override Points

| Method | Called When |
|---|---|
| `InsertItem(int index, T item)` | `Add()` or `Insert()` |
| `RemoveItem(int index)` | `Remove()` or `RemoveAt()` |
| `SetItem(int index, T item)` | Indexer assignment `col[i] = x` |
| `ClearItems()` | `Clear()` |

---

## When to Use What

| Need | Use |
|---|---|
| Internal data storage | `List<T>` from `System.Collections.Generic` |
| UI data binding (WPF/MAUI) | `ObservableCollection<T>` |
| Expose a list publicly without allowing modification | `ReadOnlyCollection<T>` |
| Custom collection with validation/behavior | Inherit from `Collection<T>` |
| List + dictionary lookup by embedded key | Inherit from `KeyedCollection<TKey, TItem>` |

---

![[Pasted image 20240617102350.png|center]]