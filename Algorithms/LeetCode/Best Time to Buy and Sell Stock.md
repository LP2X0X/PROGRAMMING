---
tags: algorithms, leetcode
---

```csharp
int MaxProfit(int[] prices)
{
    int minPrice = int.MaxValue;
    int maxProfit = 0;

    foreach (int price in prices)
    {
        if (price < minPrice)
            minPrice = price;
        else
            maxProfit = Math.Max(maxProfit, price - minPrice);
    }

    return maxProfit;
}```