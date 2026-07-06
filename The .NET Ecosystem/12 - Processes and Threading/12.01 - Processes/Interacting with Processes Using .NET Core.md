---
tags:
 - csharp
 - processes
---

# Interacting with Processes Using .NET

The `System.Diagnostics` namespace provides types for programmatically working with processes — inspecting what's running, starting new processes, and terminating existing ones. The central class is `System.Diagnostics.Process`.


---

## The Process Class — Key Capabilities

`Process` lets you do four things:

1. **Enumerate** running processes on the local (or remote) machine
2. **Inspect** a process — its threads, loaded modules, memory usage, priority
3. **Start** new processes programmatically
4. **Kill** running processes

```csharp
using System.Diagnostics;
```


---

## Enumerating Running Processes

```csharp
// All processes on this machine
Process[] allProcs = Process.GetProcesses();
foreach (Process proc in allProcs)
{
    Console.WriteLine($"PID: {proc.Id,-8} Name: {proc.ProcessName}");
}

// All processes on a remote machine
Process[] remoteProcs = Process.GetProcesses("remoteMachineName");

// Find processes by name
Process[] notepadInstances = Process.GetProcessesByName("notepad");
Console.WriteLine($"Notepad instances: {notepadInstances.Length}");

// Get a specific process by PID
Process specific = Process.GetProcessById(1234);
```


---

## Inspecting a Process

Once you have a `Process` object, you can read detailed information:

```csharp
Process proc = Process.GetCurrentProcess();

// Basic info
Console.WriteLine($"ID:              {proc.Id}");
Console.WriteLine($"Name:            {proc.ProcessName}");
Console.WriteLine($"Start Time:      {proc.StartTime}");
Console.WriteLine($"Priority:        {proc.BasePriority}");
Console.WriteLine($"Priority Class:  {proc.PriorityClass}");

// Memory usage
Console.WriteLine($"Working Set:     {proc.WorkingSet64 / 1024} KB");
Console.WriteLine($"Private Memory:  {proc.PrivateMemorySize64 / 1024} KB");
Console.WriteLine($"Virtual Memory:  {proc.VirtualMemorySize64 / 1024} KB");

// Threads (OS-level threads, not .NET Task/Thread objects)
ProcessThreadCollection threads = proc.Threads;
Console.WriteLine($"Thread count:    {threads.Count}");
foreach (ProcessThread t in threads)
{
    Console.WriteLine($"  Thread ID: {t.Id}, State: {t.ThreadState}");
}

// Loaded modules (DLLs and the EXE itself)
ProcessModuleCollection modules = proc.Modules;
Console.WriteLine($"Module count:    {modules.Count}");
foreach (ProcessModule mod in modules)
{
    Console.WriteLine($"  {mod.ModuleName} → {mod.FileName}");
}
```

```ad-note
`ProcessThread` and `ProcessModule` are **diagnostic** types for inspecting what the OS sees. `ProcessThread` is **not** the same as `System.Threading.Thread` — you cannot use it to create or manage threads. See [[ProcessThread]] and [[Process's Module Set]].
```


---

## Starting a New Process

### Simple — Just Launch It

```csharp
// Open Notepad
Process.Start("notepad.exe");

// Open a file with its default application
Process.Start(new ProcessStartInfo("report.pdf") { UseShellExecute = true });

// Open a URL in the default browser
Process.Start(new ProcessStartInfo("https://learn.microsoft.com") { UseShellExecute = true });
```

### With Full Control — ProcessStartInfo

For fine-grained control over how the process starts, use [[Controlling Process Startup Using the ProcessStartInfo|ProcessStartInfo]]:

```csharp
var startInfo = new ProcessStartInfo
{
    FileName = "dotnet",
    Arguments = "run --project MyApp",
    UseShellExecute = false,
    RedirectStandardOutput = true,
    RedirectStandardError = true,
    CreateNoWindow = true,
};

using Process proc = Process.Start(startInfo)!;

string output = proc.StandardOutput.ReadToEnd();
string errors = proc.StandardError.ReadToEnd();
proc.WaitForExit();

Console.WriteLine($"Exit code: {proc.ExitCode}");
Console.WriteLine(output);
```

### Key ProcessStartInfo Properties

| Property | What it controls |
|---|---|
| `FileName` | The executable or document to launch |
| `Arguments` | Command-line arguments |
| `UseShellExecute` | `true` = OS shell opens it (supports URLs, file associations). `false` = direct execution (required for stream redirection) |
| `RedirectStandardOutput` | Capture the process's stdout |
| `RedirectStandardError` | Capture the process's stderr |
| `RedirectStandardInput` | Write to the process's stdin |
| `CreateNoWindow` | Launch without a visible console window |
| `WorkingDirectory` | Set the working directory for the process |
| `Verb` | The shell [[Verb]] to use (e.g., "runas" for admin, "print") |


---

## Modifying a Process's Priority

You can change how much CPU time the OS scheduler gives a process:

```csharp
Process proc = Process.GetCurrentProcess();

// Raise priority (use with caution)
proc.PriorityClass = ProcessPriorityClass.AboveNormal;

// Lower priority (good for background work)
proc.PriorityClass = ProcessPriorityClass.BelowNormal;
```

| Priority Class | Use Case |
|---|---|
| `RealTime` | Extreme — can starve other processes (avoid) |
| `High` | Time-critical work |
| `AboveNormal` | More important than average |
| `Normal` | Default |
| `BelowNormal` | Background work |
| `Idle` | Only runs when CPU is otherwise idle |

```ad-warning
Setting a process to `RealTime` priority can make the system unresponsive. It is almost never appropriate for application code.
```


---

## Killing a Process

```csharp
// Kill by reference
Process[] notepad = Process.GetProcessesByName("notepad");
foreach (Process proc in notepad)
{
    proc.Kill();       // Immediately terminates — no cleanup, no save prompts
    proc.WaitForExit(); // Wait for the OS to finish tearing it down
}

// Kill with a timeout (wait up to 5 seconds)
proc.Kill();
if (!proc.WaitForExit(5000))
{
    Console.WriteLine("Process did not exit in time");
}
```

```ad-danger
`Kill()` forcefully terminates the process. It does **not** give the process a chance to save data, release resources, or run shutdown handlers. Use `CloseMainWindow()` first for GUI applications to request a graceful shutdown:

```csharp
proc.CloseMainWindow(); // Sends WM_CLOSE — like clicking the X button
if (!proc.WaitForExit(3000))
{
    proc.Kill(); // Force kill if it didn't close gracefully
}
```


---

## Waiting for a Process to Exit

```csharp
using Process proc = Process.Start("dotnet", "build")!;

// Block until the process finishes
proc.WaitForExit();
Console.WriteLine($"Exited with code: {proc.ExitCode}");

// Or wait with a timeout
if (proc.WaitForExit(10_000))
    Console.WriteLine($"Finished: exit code {proc.ExitCode}");
else
    Console.WriteLine("Still running after 10 seconds");

// Or use the async version
await proc.WaitForExitAsync();
```


---

## Subscribing to Process Events

You can be notified when a process exits without blocking:

```csharp
var proc = new Process
{
    StartInfo = new ProcessStartInfo("notepad.exe"),
    EnableRaisingEvents = true,  // must be true for events to fire
};

proc.Exited += (sender, e) =>
{
    Console.WriteLine($"Notepad exited with code: {proc.ExitCode}");
};

proc.Start();

// Your app continues running — the Exited event fires when notepad closes
```

```ad-note
`EnableRaisingEvents` must be set to `true` **before** calling `Start()`, otherwise the `Exited` event will never fire.
```
