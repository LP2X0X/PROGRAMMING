---
tags:
  - algorithms
  - leetcode
---

```csharp
// O(n) time
int[] TwoSum(int[] nums, int target)
{
    var seen = new Dictionary<int, int>();

    for (int index = 0; index < nums.Length; index++)
    {
        var diff = target - nums[index];
        if (seen.ContainsKey(diff))
        {
            return [seen[diff], index];
        }
        else
        {
            seen.Add(nums[index], index);
        }
    }
}
```