---
tags: algorithms, leetcode
---

```csharp
ListNode MergeTwoLists(ListNode list1, ListNode list2)
{
    ListNode dummy = new();
    ListNode tail = dummy;

    while (list1 != null && list1 != null)
    {
        if (list1.val < list2.val)
        {
            tail.next = list1;
            list1 = list1.next;
        }
        else
        {
            tail.next = list2;
            list2 = list2.next;
        }

        tail = tail.next;
    }

    tail = list1 ?? list2;

    return dummy.next;
}

public class ListNode
{
    public int val;
    public ListNode next;
    public ListNode(int val = 0, ListNode next = null)
    {
        this.val = val;
        this.next = next;
    }
}
```