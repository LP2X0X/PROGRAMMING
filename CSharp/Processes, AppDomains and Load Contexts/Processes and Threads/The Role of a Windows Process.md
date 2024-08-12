- The concept of a “process” existed within Windows-based operating systems well before the release of the .NET/.NET Core platforms.
- In simple terms, a *process* is a running program. However, formally speaking, a process is an operating system-level concept used to describe a set of resources (such as external code libraries and the primary thread) and the necessary memory allocations used by a running application.

```ad-note
For each .NET Core application loaded into memory, the OS creates a separate and isolated process for use during its lifetime.
```
- Using this approach to application isolation, the result is a much more robust and stable runtime environment, given that the failure of one process does not affect the functioning of another. Furthermore, data in one process cannot be directly accessed by another process, unless you use specific tools such as System.IO.Pipes or the MemoryMappedFile class. Given these points, you can regard the process as a fixed, safe boundary for a running application.

```ad-note
Every Windows process is assigned a unique process identifier (PID) and may be independently loaded and unloaded by the OS as necessary (as well as programmatically).
```