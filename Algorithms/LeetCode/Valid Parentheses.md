---
tags:
  - algorithms
  - leetcode
---

```csharp
bool IsValid(string s)
{
    Stack<char> stack = new();

    var map = new Dictionary<char, char>()
    {
        {')' , '('},
        {'}' , '{'},
        {']' , '['},
    };

    foreach (char c in s)
    {
        if (map.ContainsKey(c))
        {
            char top = stack.Count > 0 ? stack.Pop() : '_'; // Check if there is no open bracket
            if (top != map[c]) // Check if mismatch bracket
            {
                return false;
            }
        }
        else
        {
            stack.Push(c);
        }
    }
}
```