---
tags: algorithms, grokking, basic
---

### Summary
- Binary search is an algorithm; its input is a **sorted** list of elements. 
	- If an element you’re looking for is in that list, binary search returns the position where it’s located.
	- Otherwise, binary search returns null.
- In general, for any list of n, binary search will take log<sub>2</sub>(n) steps to run in the worst case, whereas simple search will take n steps.

### ✅ Solution

#### 1. Rust
```rust
fn binary_search(arr: &[isize], item: isize) -> Option<usize> {
    let mut low = 0;
    let mut high = arr.len() - 1;

    while low <= high {
        let mid = (low + high) / 2;
        let guess = arr[mid];
        if arr[mid] == item {
            return Some(mid);
        } else if guess < item {
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return None;
}
```

#### 2. JavaScript
```js
function binary_search(arr, item) {
  let low = 0;
  let high = arr.length - 1;

  while (low <= high) {
    let mid = Math.floor((low + high) / 2);
    let guess = arr[mid];

    if (guess == item) {
      return mid;
    } else if (guess < item) {
      low = mid + 1;
    } else {
      high = mid - 1;
    }
  }

  return undefined;
}
```

#### 3. C++

```cpp
#include <iostream>
#include <cmath>
#include <vector>

int binary_search(const std::vector<int> &arr, int item);

// preferred version
int main()
{
}

int binary_search(const std::vector<int> &arr, int item)
{
	int low = 0;
	int high = static_cast<int>(arr.size()) - 1;

	while (low <= high)
	{
		int mid = (low + high) / 2;
		int guess = arr[mid];

		if (guess == item)
		{
			return mid;
		}
		else if (guess < item)
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