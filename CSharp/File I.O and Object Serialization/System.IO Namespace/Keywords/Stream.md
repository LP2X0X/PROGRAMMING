- In the world of I/O manipulation, a **stream** represents a chunk of data flowing between a source and a destination. Streams provide a common way to interact with a sequence of **bytes**, regardless of what kind of device (e.g., file, network connection, or printer) stores or displays the bytes in question.
- The abstract System.IO.Stream class defines several members that provide support for synchronous and asynchronous interactions with the storage medium (e.g., an underlying file or memory location).
- Some Stream-derived types support [[Seeking|seeking]], which refers to the process of obtaining and adjusting the current position in the stream.

```ad-note
The concept of a stream is not limited to file I/O. To be sure, the .neT libraries provide stream access to networks, memory locations, and other stream-centric abstractions.
```

![[Pasted image 20240723095044.png|center|700]]