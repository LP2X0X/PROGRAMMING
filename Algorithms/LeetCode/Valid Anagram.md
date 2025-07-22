---
tags: algorithms, leetcode
---

```csharp
bool IsAnagram(string s, string t)
{
    if (s.Length != t.Length) return false;
    int[] count = new int[26];

    for (int i = 0; i < s.Length; i++)
    {
        count[s[i] - 'a']++;
        count[t[i] - 'a']--;
    }

    return count.All(x => x == 0);
}
```