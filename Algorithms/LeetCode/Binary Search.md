---
tags: algorithms, leetcode
---

```csharp
int Search(int[] nums, int target)
{
    var low = 0;
    var high = nums.Length - 1;

    while (low <= high)
    {
        var mid = (low + high) / 2;
        var guess = nums[mid];

        if (guess == target)
        {
            return mid;
        }
        else if (guess < target)
        {
            low = mid + 1;
        }
        else
        {
            high = mid - 1;
        }
    }
    return -1;
}
```