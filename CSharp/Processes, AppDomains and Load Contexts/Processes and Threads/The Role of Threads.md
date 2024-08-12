- Every Windows process contains an initial “thread” that functions as the entry point for the application.
- A **thread** is a path of execution within a process. Formally speaking, the first thread created by a process’s entry point is termed the primary thread.

```ad-note
Processes that contain a single primary thread of execution are intrinsically thread-safe, given that there is only one thread that can access the data in the application at a given time.
```

- Given the potential drawback of single-threaded applications, the operating systems that are supported by .NET Core (as well as the .NET Core platform) make it possible for the primary thread to spawn additional secondary threads (also termed worker threads) using a handful of API functions such as CreateThread(). Each thread (primary or secondary) becomes a unique path of execution in the process and has concurrent access to all shared points of data within the process.