---
tags:
 - csharp
 - collections
 - generics
---

## What Is a Stack?

A `Stack<T>` is a collection that maintains items in a **LIFO (Last-In, First-Out)** order. The last item you put in is the first item you take out — like a stack of plates.

```
Push("A") → Push("B") → Push("C")

  ┌───┐
  │ C │  ← top (last in, first out)
  ├───┤
  │ B │
  ├───┤
  │ A │  ← bottom (first in, last out)
  └───┘

Pop() → returns "C"
Pop() → returns "B"
Pop() → returns "A"
```

---

## Key Methods

| Method                  | Description                                                                 |
| ----------------------- | --------------------------------------------------------------------------- |
| `Push(T item)`          | Add item to the top of the stack                                            |
| `Pop()`                 | Remove and return the top item. Throws `InvalidOperationException` if empty |
| `Peek()`                | Return the top item without removing it. Throws if empty                    |
| `TryPop(out T result)`  | Try to pop — returns `false` instead of throwing if empty                   |
| `TryPeek(out T result)` | Try to peek — returns `false` instead of throwing if empty                  |
| `Count`                 | Number of items currently in the stack                                      | 
| `Contains(T item)`      | Check if an item exists in the stack                                        |
| `Clear()`               | Remove all items                                                            |
| `ToArray()`             | Copy stack to an array (top of stack = index 0)                             |

---

## Basic Usage

```csharp
Stack<int> stack = new Stack<int>();

stack.Push(10);
stack.Push(20);
stack.Push(30);

Console.WriteLine(stack.Peek());  // 30 — look at top without removing
Console.WriteLine(stack.Pop());   // 30 — removes and returns top
Console.WriteLine(stack.Pop());   // 20
Console.WriteLine(stack.Count);   // 1 — only 10 left

// Safe access — no exception if empty
if (stack.TryPop(out int val))
    Console.WriteLine(val);       // 10
if (stack.TryPop(out int val2))
    Console.WriteLine(val2);      // doesn't execute — stack is empty
```

---

## Example with Objects

```csharp
static void UseGenericStack()  
{  
	Stack<Person> stackOfPeople = new();  
	stackOfPeople.Push(new Person { FirstName = "Homer", LastName = "Simpson", Age = 47 });  
	stackOfPeople.Push(new Person { FirstName = "Marge", LastName = "Simpson", Age = 45 });  
	stackOfPeople.Push(new Person { FirstName = "Lisa", LastName = "Simpson", Age = 9 });  
	// Now look at the top item, pop it, and look again.  
	Console.WriteLine("First person is: {0}", stackOfPeople.Peek());  
	Console.WriteLine("Popped off {0}", stackOfPeople.Pop());  
	Console.WriteLine("\nFirst person is: {0}", stackOfPeople.Peek());  
	Console.WriteLine("Popped off {0}", stackOfPeople.Pop());  
	Console.WriteLine("\nFirst person item is: {0}", stackOfPeople.Peek());  
	Console.WriteLine("Popped off {0}", stackOfPeople.Pop());  
	try  
	{  
		Console.WriteLine("\nnFirst person is: {0}", stackOfPeople.Peek());  
		Console.WriteLine("Popped off {0}", stackOfPeople.Pop());  
	}  
	catch (InvalidOperationException ex)
	{  
		Console.WriteLine("\nError! {0}", ex.Message);  
	}  
}

// Output:
// First person is: Lisa Simpson
// Popped off Lisa Simpson
//
// First person is: Marge Simpson
// Popped off Marge Simpson
//
// First person item is: Homer Simpson
// Popped off Homer Simpson
//
// Error! Stack empty.
```

---

## Iterating a Stack

You can `foreach` over a stack (top to bottom), but you **cannot** access by index:

```csharp
Stack<string> stack = new Stack<string>();
stack.Push("A");
stack.Push("B");
stack.Push("C");

foreach (string item in stack)
    Console.Write(item + " ");  // C B A — top to bottom

// stack[0]  ← compile error — no indexer on Stack<T>
```

---

## Common Use Cases

- **Undo/Redo** — push actions, pop to undo
- **Browser back button** — push visited pages, pop to go back
- **DFS (Depth-First Search)** — traverse graphs/trees
- **Expression parsing** — balancing parentheses, evaluating postfix
- **Call stack** — how method calls work internally in the CLR

---

## Time Complexity

| Operation | Time           |
| --------- | -------------- |
| Push      | O(1) amortized |
| Pop       | O(1)           |
| Peek      | O(1)           |
| Contains  | O(n)           |