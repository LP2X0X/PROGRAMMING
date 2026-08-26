---
tags:
  - bit-manipulation
  - technique
  - xor
  - algorithm
---

## Find the Lone Non-Duplicate

A classic algorithmic problem: given an array where every element appears **exactly twice** except for one unique element, find that element. The bit manipulation solution using the [[XOR Operator]] achieves O(n) time and O(1) space -- optimal on both fronts.

---

## The Problem

```
Input:  [4, 1, 2, 1, 2]
Output: 4

Input:  [2, 2, 1]
Output: 1

Input:  [7, 3, 5, 3, 7]
Output: 5
```

Every element appears twice, canceling out, except for the lone unique element.

---

## The XOR Solution

**Algorithm:** XOR all elements together. The result is the unique element.

```
result = 0
for each element in array:
    result ^= element
return result
```

That is it. Three lines.

---

## Why It Works

The solution relies on three fundamental properties of the [[XOR Operator]]:

1. **Self-inverse:** `a ^ a = 0` -- any value XOR itself cancels to zero
2. **Identity:** `a ^ 0 = a` -- XOR with zero preserves the value
3. **Commutativity and Associativity:** the order of XOR operations does not matter

Since XOR is commutative and associative, we can mentally regroup the terms:

```
[4, 1, 2, 1, 2]

XOR all:  4 ^ 1 ^ 2 ^ 1 ^ 2

Regroup:  (1 ^ 1) ^ (2 ^ 2) ^ 4     (commutative + associative)
        =    0    ^    0    ^ 4       (self-inverse: a ^ a = 0)
        =              0   ^ 4       (identity: 0 ^ 0 = 0)
        =                    4       (identity: 0 ^ a = a)
```

All duplicates cancel to zero. Only the unique element survives.

---

## Step-by-Step Walkthrough

**Array: [7, 3, 5, 3, 7]** -- Expected result: 5

```
Start:    result = 00000000    (0)

Step 1:   result ^= 7
          00000000
        ^ 00000111
        ----------
          00000111             result = 7

Step 2:   result ^= 3
          00000111
        ^ 00000011
        ----------
          00000100             result = 4

Step 3:   result ^= 5
          00000100
        ^ 00000101
        ----------
          00000001             result = 1

Step 4:   result ^= 3
          00000001
        ^ 00000011
        ----------
          00000010             result = 2

Step 5:   result ^= 7
          00000010
        ^ 00000111
        ----------
          00000101             result = 5

Final:    result = 5           the lone non-duplicate
```

```ad-tip
title: Order Does Not Matter
Notice how we processed elements in their original order, not sorted or grouped. XOR's commutativity guarantees the same result regardless of element ordering.
```

---

## Complexity Analysis

| Metric | XOR Solution | Hash Set Solution    |
| ------ | ------------ | -------------------- |
| Time   | O(n)         | O(n)                 |
| Space  | **O(1)**     | O(n)                 |

The hash set approach adds elements if unseen and removes them if already present. After processing all elements, the remaining element is the unique one. It uses O(n) extra space, while the XOR approach uses only a single variable.

---

## Code Examples

### C\#

```csharp
public static class LoneElement
{
    /// <summary>
    /// Find the element that appears exactly once.
    /// All other elements appear exactly twice.
    /// </summary>
    public static int FindUnique(int[] nums)
    {
        int result = 0;
        foreach (int num in nums)
        {
            result ^= num;
        }
        return result;
    }
}

// Usage
int[] arr1 = { 4, 1, 2, 1, 2 };
Console.WriteLine(LoneElement.FindUnique(arr1));  // 4

int[] arr2 = { 2, 2, 1 };
Console.WriteLine(LoneElement.FindUnique(arr2));  // 1

int[] arr3 = { 7, 3, 5, 3, 7 };
Console.WriteLine(LoneElement.FindUnique(arr3));  // 5

// Works with negative numbers too
int[] arr4 = { -1, 5, -1, 3, 5 };
Console.WriteLine(LoneElement.FindUnique(arr4));  // 3

// LINQ one-liner alternative
int unique = arr1.Aggregate(0, (acc, x) => acc ^ x);
Console.WriteLine(unique);  // 4
```

### C++

```cpp
#include <iostream>
#include <vector>
#include <numeric>  // std::accumulate

// Find the element that appears exactly once
int findUnique(const std::vector<int>& nums) {
    int result = 0;
    for (int num : nums) {
        result ^= num;
    }
    return result;
}

// Alternative using std::accumulate
int findUniqueStl(const std::vector<int>& nums) {
    return std::accumulate(nums.begin(), nums.end(), 0,
                           [](int acc, int x) { return acc ^ x; });
}

int main() {
    std::vector<int> arr1 = {4, 1, 2, 1, 2};
    std::cout << findUnique(arr1) << "\n";  // 4

    std::vector<int> arr2 = {7, 3, 5, 3, 7};
    std::cout << findUnique(arr2) << "\n";  // 5

    std::vector<int> arr3 = {-1, 5, -1, 3, 5};
    std::cout << findUnique(arr3) << "\n";  // 3

    return 0;
}
```

### JavaScript

```javascript
// Find the element that appears exactly once
function findUnique(nums) {
    let result = 0;
    for (const num of nums) {
        result ^= num;
    }
    return result;
}

// Alternative using reduce
function findUniqueReduce(nums) {
    return nums.reduce((acc, x) => acc ^ x, 0);
}

// Usage
console.log(findUnique([4, 1, 2, 1, 2]));    // 4
console.log(findUnique([2, 2, 1]));            // 1
console.log(findUnique([7, 3, 5, 3, 7]));      // 5
console.log(findUnique([-1, 5, -1, 3, 5]));    // 3

// One-liner
const unique = [4, 1, 2, 1, 2].reduce((a, b) => a ^ b, 0);
console.log(unique);  // 4
```

---

## Extension: Finding Two Unique Elements

**Problem:** Every element appears twice except for **two** elements. Find both.

**Approach:** XOR all elements to get `xorAll = a ^ b` (the XOR of the two unique values). Since `a != b`, at least one bit in `xorAll` is `1`. Use that distinguishing bit to partition all elements into two groups, then XOR each group separately.

```
Algorithm:
  1. XOR all elements -> xorAll = a ^ b
  2. Find any set bit in xorAll (use n & (-n) to isolate the lowest)
  3. Partition elements by whether that bit is set or clear
  4. XOR each partition -> one yields a, the other yields b
```

```
Example: [1, 2, 1, 3, 2, 5]
  Unique elements: 3 and 5

Step 1: XOR all = 1^2^1^3^2^5 = 3^5 = 011^101 = 110 (6)

Step 2: Lowest set bit of 6 = 6 & (-6) = 010 (bit 1)

Step 3: Partition by bit 1:
  Bit 1 SET:   [2, 3, 2]     -> 2^3^2 = 3
  Bit 1 CLEAR: [1, 1, 5]     -> 1^1^5 = 5

Step 4: Results: 3 and 5
```

The [[Isolate and Clear Lowest Set Bit]] operation (`n & (-n)`) gives us the distinguishing bit.

```csharp
public static (int, int) FindTwoUnique(int[] nums)
{
    // Step 1: XOR all to get a ^ b
    int xorAll = 0;
    foreach (int num in nums)
        xorAll ^= num;

    // Step 2: Isolate any set bit (lowest set bit)
    int diffBit = xorAll & (-xorAll);

    // Step 3-4: Partition and XOR
    int a = 0, b = 0;
    foreach (int num in nums)
    {
        if ((num & diffBit) != 0)
            a ^= num;
        else
            b ^= num;
    }

    return (a, b);
}
```

---

## Extension: Element Appearing Once, Others Three Times

**Problem:** Every element appears **three times** except one. Find the unique element.

XOR alone cannot solve this because `a ^ a ^ a = a` (not zero). Instead, count bits at each position modulo 3.

```
Algorithm:
  1. For each bit position 0..31:
     a. Count how many numbers have that bit set
     b. If count % 3 != 0, the unique number has that bit set
  2. Reconstruct the unique number from the bit counts
```

```
Example: [2, 2, 3, 2]     (unique = 3)

  Bit 0:  0+0+1+0 = 1   -> 1 % 3 = 1  -> bit 0 is SET
  Bit 1:  1+1+1+1 = 4   -> 4 % 3 = 1  -> bit 1 is SET

  Result: bit 1 and bit 0 set = 0b11 = 3
```

```csharp
public static int FindUniqueAmongTriples(int[] nums)
{
    int result = 0;
    for (int i = 0; i < 32; i++)
    {
        int bitCount = 0;
        foreach (int num in nums)
        {
            if ((num & (1 << i)) != 0)
                bitCount++;
        }

        if (bitCount % 3 != 0)
            result |= (1 << i);
    }
    return result;
}
```

```ad-note
title: Generalization
The mod-3 bit counting approach generalizes: if every duplicate appears k times, use `bitCount % k != 0` to identify the unique element's bits. This works for any k where only one element has a different frequency.
```

---

## Edge Cases

| Scenario                    | Behavior                                |
| --------------------------- | --------------------------------------- |
| Single-element array        | Returns that element (`0 ^ a = a`)      |
| Array with negative numbers | Works correctly (XOR operates on bits)  |
| Array with zero as unique   | Works correctly (`a ^ a = 0`, `0 ^ 0 = 0`) |
| Large array                 | Still O(n) time, O(1) space             |

```ad-warning
title: Assumes Exactly One Unique Element
The XOR approach assumes **every** other element appears exactly twice. If any element appears an odd number of times, it will contaminate the result. Validate input constraints before applying this technique.
```

---

## Related Concepts

- **[[XOR Operator]]** -- the algebraic properties (self-inverse, commutativity, associativity) that make this work.
- **[[AND Operator]]** -- used in the two-unique and triple-occurrence extensions for bit checking.
- **[[Isolate and Clear Lowest Set Bit]]** -- `n & (-n)` is used to find the distinguishing bit in the two-unique extension.

---

## Summary

| Variant                    | Time  | Space | Technique                         |
| -------------------------- | ----- | ----- | --------------------------------- |
| One unique (others x2)     | O(n)  | O(1)  | XOR all elements                  |
| Two unique (others x2)     | O(n)  | O(1)  | XOR all, partition by diff bit    |
| One unique (others x3)     | O(n)  | O(1)  | Count bits mod 3                  |
| Hash set (general)         | O(n)  | O(n)  | Track seen elements               |
