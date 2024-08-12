- When the runtime is attempting to locate unreachable objects, it does not literally examine every object placed on the managed heap. Doing so, obviously, would involve considerable time, especially in larger (i.e., real-world) applications.
- To help optimize the process, each object on the heap is assigned to a specific “generation.” The idea behind generations is simple: the longer an object has existed on the heap, the more likely it is to stay there.
- Each object on the heap belongs to a collection in one of the following generations:  
	- **Generation 0**: Identifies a newly allocated object that has never been marked for collection (***with the exception of large objects, which are initially placed in a generation 2 collection***). Most objects are reclaimed for garbage collection in generation 0 and do now survive to generation 1.  
	- **Generation 1**: Identifies an object that has survived a garbage collection. This generation also serves as a buffer between short-lived objects and long-lived objects.  
	- G**eneration 2**: Identifies an object that has survived more than one sweep of the garbage collector or a significantly large object that started in a generation 2 collection.

---

- The garbage collector will investigate all generation 0 objects first. If marking and sweeping (or said more plainly, getting rid of) these objects results in the required amount of free memory, any surviving objects are promoted to generation 1.
- If all generation 0 objects have been evaluated but additional memory is still required, generation 1 objects are then investigated for reachability and collected accordingly. Surviving generation 1 objects are then promoted to generation 2. If the garbage collector still requires additional memory, generation 2 objects are evaluated. At this point, if a generation 2 object survives a garbage collection, it remains a generation 2 object, given the predefined upper limit of object generations.

```ad-note
The bottom line is that by assigning a generational value to objects on the heap, newer objects (such as local variables) will be removed quickly, while older objects (such as a program’s main window) are not “bothered” as often.
```

- Garbage collection is triggered when the system has low physical memory, when memory allocated on the managed heap rises above an acceptable threshold, or when GC.Collect() is called in the application code.

```ad-warning
If this all seems a bit wonderful and better than having to manage memory yourself, remember that the process of garbage collection is not without some cost. The timing of garbage collection and what gets collected when are typically out of the developers’ controls, although garbage collection can certainly be influenced for good or bad. And when garbage collection is executing, CPU cycles are being used and can affect the performance of the application.
```