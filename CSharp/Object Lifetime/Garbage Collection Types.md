- There are two types of garbage collection provided by the runtime:
	 - **Workstation garbage collection**: This is designed for client applications and is the default for stand-alone applications. Workstation GC can be background (covered next) or nonconcurrent.
	 - **Server garbage collection**: This is designed for server applications that require high throughput and scalability. Server GC can be background or nonconcurrent, just like workstation GC.

```ad-note
The names are indicative of the default settings for workstation and server applications, but the method of garbage collection is configurable through the machine’s runtimeconfig.json or system environment variables. Unless the computer has only one processor, then it will always use workstation garbage collection.
```