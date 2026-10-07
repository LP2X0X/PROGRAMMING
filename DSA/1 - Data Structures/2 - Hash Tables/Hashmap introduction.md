A **HashMap** is a data structure that stores key-value pairs and offers efficient insertion, deletion, and lookup operations, typically in **constant time (O(1))** on average. It is widely used in programming for fast data retrieval.

### Key Concepts

1. **Key-Value Pair**:
   - Each item in a hashmap is stored as a pair — a **key** and its associated **value**.
   - Keys are unique; values can be duplicated.

2. **Hash Function**:
   - A hash function takes a key and computes an integer called a **hash code**.
   - This hash code is then used (typically modulo the size of the array) to determine the index where the key-value pair should be stored.

3. **Buckets and Collision Handling**:
   - Hash collisions occur when two keys hash to the same index.
   - Common collision handling techniques:
     - **Chaining**: Each bucket holds a list (chain) of entries. Multiple key-value pairs can be stored in the same bucket.
     - **Open Addressing**: When a collision occurs, the algorithm searches for the next available slot according to a probing sequence.

4. **Load Factor and Rehashing**:
   - The **load factor** is the ratio of the number of stored entries to the hashmap’s capacity.
   - When the load factor exceeds a certain threshold (e.g., 0.75), **rehashing** occurs: the internal array size is increased and all entries are moved to new positions based on their new hash codes.

5. **Performance**:
   - **Time Complexity** (Average Case):
     - Insert: O(1)
     - Lookup: O(1)
     - Delete: O(1)
   - **Worst Case**: O(n) — happens if many collisions occur and all entries end up in one bucket (like a linked list).

### Use Cases

- Quickly checking if a key exists.
- Implementing associative arrays or dictionaries.
- Counting frequencies (e.g., word frequency in a document).
- Implementing caches or lookup tables.

### Example (in pseudocode):

```plaintext
hashmap = {}
hashmap["apple"] = 3
hashmap["banana"] = 7
print(hashmap["apple"])  # outputs 3
```

### Variants by Language

- **Java**: `HashMap<K, V>` class in java.util
- **Python**: `dict` type
- **C++**: `std::unordered_map`
- **JavaScript**: `Map` object

In summary, a **hashmap** is a powerful and versatile data structure used to efficiently store and retrieve data based on keys, and is an essential tool in most modern programming languages.