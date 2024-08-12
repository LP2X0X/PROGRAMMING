- The garbage collector uses the following information to determine whether an object is live:
	-	 Stack roots: Stack variables provided by the compiler and stack walker
	-	 Garbage collection handles: Handles that point to managed objects that can be referenced from code or the runtime
	-	 Static data: Static objects in application domains that can reference other objects
- During a garbage collection process, the runtime will investigate objects on the managed heap to determine whether they are still reachable by the application. To do so, the runtime will build an object graph, which represents each reachable object on the heap. Object graphs are used to document all reachable objects. As well, be aware that the garbage collector will never graph the same object twice, thus avoiding the nasty circular reference count found in COM programming.
![[Pasted image 20240605093217.png|center]]
![[Pasted image 20240605093228.png|center]]

```ad-note
Strictly speaking, the garbage collector uses two distinct heaps, one of which is specifically used to store large objects. This heap is less frequently consulted during the collection cycle, given possible performance penalties involved with relocating large objects. in .NET Core, the large heap can be compacted on demand or when optional hard limits for absolute or percentage memory usage is reached.
 ```


