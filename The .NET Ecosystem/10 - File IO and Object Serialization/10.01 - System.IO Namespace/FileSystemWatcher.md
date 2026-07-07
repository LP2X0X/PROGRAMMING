---
tags:
 - csharp
 - io
 - filesystemwatcher
---

## FileSystemWatcher

`FileSystemWatcher` monitors a directory for file system changes and raises events in real time when files or subdirectories are created, modified, deleted, or renamed. It is an event-driven alternative to polling a directory on a timer, and it is the foundation for features like "hot reload", file sync, and live log tailing.

---

## Basic Setup

```csharp
using System.IO;

var watcher = new FileSystemWatcher(@"C:\WatchDir")
{
    // Which files to watch (default "*.*" = all files)
    Filter = "*.txt",

    // What changes to listen for
    NotifyFilter = NotifyFilters.LastWrite
                 | NotifyFilters.FileName
                 | NotifyFilters.DirectoryName,

    // Whether to watch subdirectories recursively
    IncludeSubdirectories = false,

    // MUST set this to true or no events will fire
    EnableRaisingEvents = true
};

// Subscribe to events
watcher.Created += (sender, e) => Console.WriteLine($"Created: {e.FullPath}");
watcher.Deleted += (sender, e) => Console.WriteLine($"Deleted: {e.FullPath}");
watcher.Changed += (sender, e) => Console.WriteLine($"Changed: {e.FullPath}");
watcher.Renamed += (sender, e) => Console.WriteLine($"Renamed: {e.OldFullPath} -> {e.FullPath}");
watcher.Error   += (sender, e) => Console.WriteLine($"Error: {e.GetException().Message}");
```

```ad-important
title: EnableRaisingEvents Must Be True
The watcher does NOT raise events by default. You must explicitly set `EnableRaisingEvents = true`. This is the most common mistake when using `FileSystemWatcher`. Without it, the watcher is "armed" but silent.
```

---

## Key Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `Path` | `string` | `""` | Directory to monitor (must exist) |
| `Filter` | `string` | `"*.*"` | File name filter (e.g., `"*.txt"`, `"report_*"`) |
| `Filters` | `Collection<string>` | empty | .NET 6+: multiple filters (`"*.txt"`, `"*.csv"`) |
| `NotifyFilter` | `NotifyFilters` | `LastWrite \| FileName \| DirectoryName` | What types of changes to detect |
| `IncludeSubdirectories` | `bool` | `false` | Watch subdirectories recursively |
| `EnableRaisingEvents` | `bool` | `false` | Master switch -- events only fire when `true` |
| `InternalBufferSize` | `int` | 8192 (8 KB) | Size of the internal buffer for change notifications |

### Multiple Filters (.NET 6+)

Before .NET 6, you could only set a single `Filter`. To watch multiple file types, you either used `"*.*"` and filtered in the event handler, or created multiple watchers. In .NET 6+, you can add multiple patterns:

```csharp
var watcher = new FileSystemWatcher(@"C:\WatchDir");
watcher.Filters.Add("*.txt");
watcher.Filters.Add("*.csv");
watcher.Filters.Add("*.log");
watcher.EnableRaisingEvents = true;
```

---

## Events

| Event | EventArgs Type | Fires When |
|---|---|---|
| `Created` | `FileSystemEventArgs` | A file or directory is created |
| `Deleted` | `FileSystemEventArgs` | A file or directory is deleted |
| `Changed` | `FileSystemEventArgs` | A file or directory is modified (size, attributes, last write, etc.) |
| `Renamed` | `RenamedEventArgs` | A file or directory is renamed |
| `Error` | `ErrorEventArgs` | The watcher encounters an error (buffer overflow, access lost) |

### FileSystemEventArgs Properties

| Property | Type | Description |
|---|---|---|
| `ChangeType` | `WatcherChangeTypes` | What kind of change occurred |
| `FullPath` | `string` | Full path of the affected file/directory |
| `Name` | `string?` | Name of the affected file/directory (relative to watched path) |

### RenamedEventArgs Additional Properties

| Property | Type | Description |
|---|---|---|
| `OldFullPath` | `string` | Previous full path before rename |
| `OldName` | `string?` | Previous name before rename |

### ErrorEventArgs

```csharp
watcher.Error += (sender, e) =>
{
    Exception ex = e.GetException();
    Console.WriteLine($"Watcher error: {ex.Message}");

    // Common: InternalBufferOverflowException when too many changes happen at once
    if (ex is InternalBufferOverflowException)
    {
        Console.WriteLine("Buffer overflow -- some file changes were missed!");
    }
};
```

---

## NotifyFilters Enum

`NotifyFilters` controls **what types of changes** trigger the `Changed` event. It is a flags enum, so you combine values with `|`.

| Value | Description |
|---|---|
| `FileName` | File name changes (create, delete, rename) |
| `DirectoryName` | Directory name changes (create, delete, rename) |
| `Attributes` | File or directory attribute changes (hidden, read-only, etc.) |
| `Size` | File size changes |
| `LastWrite` | Last write time changes (most common -- fires when file content is modified) |
| `LastAccess` | Last access time changes |
| `CreationTime` | Creation time changes |
| `Security` | Security descriptor changes (ACL modifications) |

```csharp
// Watch only for content changes and file creation/deletion
watcher.NotifyFilter = NotifyFilters.LastWrite | NotifyFilters.FileName;

// Watch for everything
watcher.NotifyFilter = NotifyFilters.Attributes
                     | NotifyFilters.CreationTime
                     | NotifyFilters.DirectoryName
                     | NotifyFilters.FileName
                     | NotifyFilters.LastAccess
                     | NotifyFilters.LastWrite
                     | NotifyFilters.Security
                     | NotifyFilters.Size;
```

```ad-note
title: NotifyFilter Affects Only the Changed Event
`NotifyFilter` only controls what triggers the `Changed` event. The `Created`, `Deleted`, and `Renamed` events always fire for file/directory operations regardless of the `NotifyFilter` setting (as long as `FileName` or `DirectoryName` is included).
```

---

## WatcherChangeTypes Enum

Available on the `ChangeType` property of `FileSystemEventArgs`:

| Value | Description |
|---|---|
| `Created` | A file or directory was created |
| `Deleted` | A file or directory was deleted |
| `Changed` | A file or directory was modified |
| `Renamed` | A file or directory was renamed |
| `All` | Any of the above (used for filtering, not reported) |

---

## Common Pitfalls and Workarounds

### Duplicate Changed Events

```ad-warning
title: Changed Event Can Fire Multiple Times for a Single Save
When many applications save a file, they perform multiple operations internally (write, flush, update attributes, update timestamps). Each operation can trigger a separate `Changed` event. For example, saving a file in Notepad might fire 2-3 `Changed` events in rapid succession.

**This is the most common surprise with FileSystemWatcher.**
```

**Workaround -- Debounce with a Timer:**

```csharp
using System;
using System.IO;
using System.Collections.Concurrent;
using System.Threading;

// Track the last event time per file to debounce
ConcurrentDictionary<string, DateTime> lastEvents = new();
TimeSpan debounceInterval = TimeSpan.FromMilliseconds(500);

var watcher = new FileSystemWatcher(@"C:\WatchDir")
{
    Filter = "*.txt",
    NotifyFilter = NotifyFilters.LastWrite | NotifyFilters.FileName,
    EnableRaisingEvents = true
};

watcher.Changed += (sender, e) =>
{
    DateTime now = DateTime.UtcNow;
    DateTime lastTime = lastEvents.GetOrAdd(e.FullPath, DateTime.MinValue);

    // Ignore if we already handled a change for this file within the debounce window
    if (now - lastTime < debounceInterval)
        return;

    lastEvents[e.FullPath] = now;
    Console.WriteLine($"Changed: {e.FullPath} at {now:HH:mm:ss.fff}");
};
```

A more robust approach uses `System.Threading.Timer` to delay processing until events settle:

```csharp
ConcurrentDictionary<string, Timer> debounceTimers = new();

watcher.Changed += (sender, e) =>
{
    // Reset or create a timer that fires 300ms after the LAST change event
    debounceTimers.AddOrUpdate(
        e.FullPath,
        // Create new timer
        path =>
        {
            return new Timer(_ =>
            {
                Console.WriteLine($"File settled: {path}");
                debounceTimers.TryRemove(path, out _);
            }, null, 300, Timeout.Infinite);
        },
        // Reset existing timer
        (path, existingTimer) =>
        {
            existingTimer.Change(300, Timeout.Infinite);
            return existingTimer;
        }
    );
};
```

### Internal Buffer Overflow

```ad-warning
title: Buffer Overflow Under High Volume
`FileSystemWatcher` uses an internal buffer (default 8 KB) to store pending change notifications from the OS. If many changes happen faster than your event handlers can process them, the buffer overflows and events are **silently lost**. The `Error` event fires with an `InternalBufferOverflowException`.
```

**Workaround -- Increase Buffer Size:**

```csharp
// Increase buffer for high-volume directories
// Each notification takes ~16 bytes (file name dependent), so:
// 8 KB  = ~500 notifications (default)
// 64 KB = ~4,000 notifications
// Max recommended = 64 KB (higher values have diminishing returns)
watcher.InternalBufferSize = 64 * 1024; // 64 KB
```

```ad-note
title: Buffer Size Tradeoffs
Increasing `InternalBufferSize` uses non-paged kernel memory that cannot be swapped to disk. Microsoft recommends keeping it at 64 KB or below. If you're still overflowing at 64 KB, your event handlers are too slow -- offload work to a background queue.
```

**Better Workaround -- Fast Handler + Background Queue:**

```csharp
using System.Collections.Concurrent;
using System.Threading.Tasks;

BlockingCollection<string> changeQueue = new();

watcher.Changed += (sender, e) =>
{
    // Do minimal work in the event handler -- just queue the path
    changeQueue.TryAdd(e.FullPath);
};

// Process changes on a separate thread
Task.Run(() =>
{
    foreach (string path in changeQueue.GetConsumingEnumerable())
    {
        // Heavy processing happens here, off the watcher thread
        ProcessFileChange(path);
    }
});
```

### Thread Safety

```ad-warning
title: Events Fire on Thread Pool Threads
`FileSystemWatcher` events fire on thread pool threads, NOT on the UI thread. If you're in a WinForms or WPF application, you must marshal back to the UI thread to update controls:

    // WinForms
    watcher.Changed += (s, e) =>
    {
        this.Invoke(() => listBox.Items.Add(e.Name));
    };
    
    // WPF
    watcher.Changed += (s, e) =>
    {
        Dispatcher.Invoke(() => fileList.Add(e.Name));
    };

Alternatively, set `watcher.SynchronizingObject = this;` in a WinForms form to have events automatically marshaled.
```

### File Locking

```ad-note
title: File May Still Be Locked When Event Fires
When you receive a `Created` or `Changed` event, the file may still be held open by the process that triggered the change. Attempting to read or copy the file immediately may throw an `IOException`. Add a brief retry loop:

    watcher.Created += async (s, e) =>
    {
        // Retry up to 5 times with 200ms delay
        for (int i = 0; i < 5; i++)
        {
            try
            {
                string content = File.ReadAllText(e.FullPath);
                Console.WriteLine($"Read {content.Length} chars from {e.Name}");
                return;
            }
            catch (IOException) when (i < 4)
            {
                await Task.Delay(200);
            }
        }
    };
```

---

## Complete Working Example

A console application that monitors a directory, handles all event types, debounces duplicate `Changed` events, and runs until the user presses a key:

```csharp
using System;
using System.Collections.Concurrent;
using System.IO;
using System.Threading;

string watchPath = @"C:\WatchDemo";
Directory.CreateDirectory(watchPath);

Console.WriteLine($"Watching: {watchPath}");
Console.WriteLine("Press any key to stop...\n");

// Debounce tracking
ConcurrentDictionary<string, DateTime> lastChanged = new();
TimeSpan debounce = TimeSpan.FromMilliseconds(500);

using var watcher = new FileSystemWatcher(watchPath)
{
    // Watch all file types
    Filter = "*.*",

    // Detect content changes, new files, deletions, renames
    NotifyFilter = NotifyFilters.LastWrite
                 | NotifyFilters.FileName
                 | NotifyFilters.DirectoryName
                 | NotifyFilters.Size,

    // Include subdirectories
    IncludeSubdirectories = true,

    // Increase buffer for safety
    InternalBufferSize = 32 * 1024,

    // Start watching
    EnableRaisingEvents = true
};

watcher.Created += (s, e) =>
    Console.WriteLine($"  [CREATED]  {e.FullPath}");

watcher.Deleted += (s, e) =>
    Console.WriteLine($"  [DELETED]  {e.FullPath}");

watcher.Changed += (s, e) =>
{
    // Debounce: ignore rapid-fire duplicates for the same file
    DateTime now = DateTime.UtcNow;
    if (lastChanged.TryGetValue(e.FullPath, out DateTime last) && now - last < debounce)
        return;
    lastChanged[e.FullPath] = now;
    Console.WriteLine($"  [CHANGED]  {e.FullPath}");
};

watcher.Renamed += (s, e) =>
    Console.WriteLine($"  [RENAMED]  {e.OldName} -> {e.Name}");

watcher.Error += (s, e) =>
{
    Exception ex = e.GetException();
    Console.WriteLine($"  [ERROR]    {ex.GetType().Name}: {ex.Message}");
};

Console.ReadKey(intercept: true);
Console.WriteLine("\nStopped watching.");
```

---

## Using WaitForChanged (Synchronous Blocking)

For simple scripts where you want to block until a specific change occurs, `FileSystemWatcher` provides a synchronous alternative:

```csharp
using var watcher = new FileSystemWatcher(@"C:\DropFolder")
{
    Filter = "*.csv"
};

Console.WriteLine("Waiting for a .csv file to appear...");

// Blocks the current thread until a matching change occurs or timeout
WaitForChangedResult result = watcher.WaitForChanged(WatcherChangeTypes.Created, timeout: 30_000);

if (result.TimedOut)
{
    Console.WriteLine("Timed out -- no file appeared within 30 seconds.");
}
else
{
    Console.WriteLine($"File created: {result.Name}");
}
```

```ad-note
title: WaitForChanged vs Event-Driven
`WaitForChanged` blocks the calling thread and is useful for simple sequential scripts. For applications that need to remain responsive (UI apps, services), always use the event-driven approach with `EnableRaisingEvents = true`.
```

---

## Disposing FileSystemWatcher

`FileSystemWatcher` implements `IDisposable`. Always dispose it when done to release the OS file system notification handle:

```csharp
// Preferred: using declaration
using var watcher = new FileSystemWatcher(path);

// Or: using block
using (var watcher = new FileSystemWatcher(path))
{
    watcher.EnableRaisingEvents = true;
    // ...
} // Disposed here -- stops watching automatically

// Setting EnableRaisingEvents = false stops events but does NOT release the handle.
// Always dispose to fully clean up.
```

---

## Summary

- `FileSystemWatcher` provides **real-time, event-driven** notification of file system changes -- no polling needed.
- Set `EnableRaisingEvents = true` or nothing works. This is the single most common mistake.
- The `Changed` event fires **multiple times** for a single file save -- debounce with a timer or timestamp check.
- Under high volume, the internal buffer overflows and events are **silently lost**. Increase `InternalBufferSize` (up to 64 KB) and offload heavy processing to a background queue.
- Events fire on **thread pool threads**, not the UI thread. Marshal to the UI thread in WinForms/WPF applications.
- Files may still be **locked** when the event fires. Use a retry loop when reading newly created/changed files.
- Always `Dispose()` the watcher when done to release OS notification handles.
