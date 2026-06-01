---
tags:
 - csharp
 - processes
---

## What Is a Verb?

A verb is the **action** you want the Windows shell to perform on a file. When you right-click a file in Explorer, the context menu shows items like "Open", "Edit", "Print", "Run as administrator" — each of those is a verb.

`ProcessStartInfo.Verb` lets you pick one of those actions programmatically instead of just opening the file.

```ad-warning
Verbs only work when `UseShellExecute = true`. Without it, the Verb property is **silently ignored** — no error, just no effect. This is a common source of "why isn't this working?"
```


---

## Common Verbs

| Verb        | Action                           | Example                                              |
| ----------- | -------------------------------- | ---------------------------------------------------- |
| `"open"`    | Open with default application    | Default when no verb is set                          |
| `"edit"`    | Open in the registered editor    | Right-click image → Edit opens Paint, not the viewer |
| `"print"`   | Send directly to default printer | Prints without opening a viewer window               |
| `"printto"` | Print to a specific printer      | Requires printer name as argument                    |
| `"runas"`   | Run as administrator             | Triggers UAC elevation prompt                        | 
| `"explore"` | Open a folder in Explorer        | Opens Explorer at that path                          |


---

## Examples

**Print a document without opening it:**

```csharp
var psi = new ProcessStartInfo(@"C:\Reports\quarterly.pdf")
{
    Verb = "print",
    UseShellExecute = true
};
Process.Start(psi);
```

**Run a program as administrator:**

```csharp
var psi = new ProcessStartInfo(@"C:\Tools\SystemConfig.exe")
{
    Verb = "runas",
    UseShellExecute = true
};

try
{
    Process.Start(psi);
}
catch (Win32Exception)
{
    // User clicked "No" on the UAC prompt
    Console.WriteLine("Elevation was cancelled.");
}
```

**Open a file for editing (not viewing):**

```csharp
var psi = new ProcessStartInfo(@"C:\Images\logo.png")
{
    Verb = "edit",
    UseShellExecute = true
};
Process.Start(psi);
// Opens in Paint (registered editor), NOT in the default image viewer
```


---

## Discovering Available Verbs

Not every file type supports every verb — it depends on what software is installed and what it registered in the Windows registry. You can query what's available:

```csharp
var psi = new ProcessStartInfo(@"C:\Reports\quarterly.pdf");

foreach (string verb in psi.Verbs)
{
    Console.WriteLine(verb);
}
// Might output: open, edit, print, printto
```

A `.txt` file might only have `open`, `print`, `printto`. A `.exe` file would have `open`, `runas`. It varies by file type and installed applications.


---

## How Verbs Are Registered

Verbs are stored in the **Windows Registry** under file type associations:

```
HKEY_CLASSES_ROOT
  └── .pdf
       └── (Default) = "AcroExch.Document"    ← file type key
  └── AcroExch.Document
       └── shell
            ├── open
            │    └── command = "C:\...\Acrobat.exe" "%1"
            ├── edit
            │    └── command = "C:\...\Acrobat.exe" /edit "%1"
            └── print
                 └── command = "C:\...\Acrobat.exe" /p "%1"
```

Each verb under `shell` defines a command that runs when that action is triggered. When you set `ProcessStartInfo.Verb = "print"`, the shell looks up this registry path and executes the associated command.


---

## Gotchas

- **Case-insensitive** — `"Print"`, `"print"`, and `"PRINT"` all work the same.
- **Unsupported verb** → `Win32Exception`. Check `psi.Verbs` first or wrap in try/catch.
- **`runas` + `UseShellExecute = true`** means you **cannot** redirect stdin/stdout/stderr. If you need elevation AND output capture, you'd need a helper process or named pipes.
- **`UseShellExecute = false`** silently ignores the Verb — this trips people up constantly.

See also: [[Controlling Process Startup Using the ProcessStartInfo]]
