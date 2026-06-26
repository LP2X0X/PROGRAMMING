---
tags:
  - algorithms
  - term
  - fundamental
---

### Context

- Your computer uses a stack internally called the *call stack*. Let's see it in action. Here's a simple function:
```python
def greet(name):
	print("hello, " + name + "!")
	greet2(name)
	print("getting ready to say bye...")
	bye()
```
- This function greets you and then calls two other functions:
```python
def greet2(name):
	print("how are you, " + name + "?")
	
def bye():
	print("ok bye! ")
```

### Call Stack in Action

- Suppose you call **greet("maggie")**. First, your computer allocates a box of memory for that function call.
![[Pasted image 20250715143231.png|center|300]]
- Now let’s use the memory. The variable name is set to "maggie". That needs to be saved in memory.
![[Pasted image 20250715143324.png|center|300]]
- Next, you print "hello, maggie!" Then you call **greet2("maggie")**. Again, your computer allocates a box of memory for this function call.
![[Pasted image 20250715143416.png|center|300]]
- Your computer is using a stack for these boxes. The second box is added on top of the first one. You print "how are you, maggie?" Then you return from the function call. When this happens, the box on top of the stack gets popped off.
![[Pasted image 20250715143741.png|center|300]]
- Now the topmost box on the stack is for the greet function, which means you returned to the greet function. When you called the greet2 function, the greet function was partially completed. This is the big idea behind this section: ***when you call a function from another function, the calling function is paused in a partially completed state***. All the values of the variables for that function are still stored on the call stack (i.e., in memory). Now that you’re done with the greet2 function, you’re back to the greet function, and you pick up where you left off. First, you print getting ready to say bye... Then you call the bye function.
![[Pasted image 20250715144018.png|center|300]]
- A box for that function is added to the top of the stack. Then you print ok bye! and return from the function call.
![[Pasted image 20250715144046.png|center|300]]
- And you’re back to the greet function. There’s nothing else to be done, so you return from the greet function, too. This stack, used to save the variables for multiple functions, is called the call stack.