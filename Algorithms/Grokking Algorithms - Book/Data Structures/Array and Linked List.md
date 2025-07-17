---
tags: algorithms, data_structure
---

### Array
- Using an array means all your tasks are stored contiguously (right next to each other) in memory. If the next memory address is occupied, then we need to ask the computer for a different chunk of memory that can fit our array. Then we must move the array there.
![[Pasted image 20250715101218.png|300]]
- Array is great if you need to access item in specific index since you know the address for every item in your array.
![[Pasted image 20250715101929.png|300]]

### Linked List
- With linked lists, your items can be anywhere in memory.
![[Pasted image 20250715101542.png|300]]
- Each item stores the address of the next item in the list. A bunch of random memory addresses are linked together. This is a type of Sequential Access.
	- **Sequential Access** means reading the elements one by one, starting with the first element.
![[Pasted image 20250715101603.png|300]]
- Linked lists are great if you’re going to read all the items one at a time: you can read one item, follow the address to the next item, and so on. But if you’re going to keep jumping around, linked lists are terrible.

- With lists, insert element in the middle is as easy as changing what the previous element points to.
![[Pasted image 20250715102654.png|500]]

--- 

### Summary
![[Pasted image 20250715102854.png|center|500]]