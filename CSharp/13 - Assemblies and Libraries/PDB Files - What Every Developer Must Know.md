---
tags:
  - csharp
  - assemblies
  - debugging
---

## What Are PDB Files?

A **PDB (Program Database) file** contains the debugging information that maps compiled IL or native code back to your original source code. Without a matching PDB, a debugger cannot show you source lines, local variable values, or meaningful call stacks — you are left staring at raw memory addresses and disassembly.

PDB files are produced by the compiler alongside the binary (`.dll` or `.exe`). They share a build-time GUID that permanently links a specific PDB to a specific binary.

```ad-warning
**PDB files are as important as source code.** If you lose the PDB for a production build, you can never debug that binary at the source level. The matching GUID is created at compile time and cannot be regenerated or changed after the fact. Every build's PDB files must be preserved.
```

---

## Private vs Public Builds

These two terms appear throughout PDB and Symbol Server documentation:

| Term             | Definition                                     | Debugging difficulty |
| ---------------- | ---------------------------------------------- | -------------------- |
| **Private build** | A build done on your development machine       | Easy — PDB is right next to the binary |
| **Public build**  | A build done on a build server / CI pipeline   | Hard — PDB must be stored and retrieved separately |

The entire Symbol Server infrastructure exists to solve the public build problem: ensuring you can always find the exact PDB that matches a production binary, no matter when it was built.

---

## What PDB Files Contain

.NET PDB files are **much simpler** than native C++ PDB files because most type and member information is already stored in .NET metadata (the assembly itself). The PDB only needs to supplement what metadata does not cover.

| Content                                   | Native C++ PDB | .NET PDB       |
| ----------------------------------------- | -------------- | -------------- |
| Function/method addresses                 | Yes            | No (in metadata) |
| Global variable names and addresses       | Yes            | No (in metadata) |
| Parameter and local variable names        | Yes            | Locals only    |
| Type data (classes, structs, definitions) | Yes            | No (in metadata) |
| Frame Pointer Omission (FPO) data         | Yes            | No             |
| Source file names and line numbers         | Yes            | **Yes**        |
| Local variable name-to-stack-offset map   | Yes            | **Yes**        |

```ad-info
Because .NET PDBs only store source mappings and local variable info, they are significantly smaller than native PDBs. This is a direct benefit of .NET's rich metadata system — the assembly is already self-describing (see [[The Role of .NET Assemblies#Assemblies Are Self-Describing]]).
```

---

## How the Debugger Finds PDB Files

The debugger matches a PDB to a binary using a **GUID** that the compiler embeds in both files at compile time. This is an exact match — if the GUIDs differ, the debugger rejects the PDB entirely.

The matching process:

1. The debugger loads a module (e.g., `MyApp.dll`).
2. It reads the **Debug Directories** section of the PE (Portable Executable) header to extract the embedded GUID and the original build path.
3. It searches for a file named `MyApp.pdb` whose internal GUID matches.

```ad-warning
The GUID is generated once at compile time and **cannot be changed or regenerated**. If you built yesterday and did not save the PDB, that binary can never be debugged at the source level again. There is no workaround.
```

---

## Debugger PDB Search Order

When the debugger needs a PDB, it searches these locations in order:

1. **The directory where the binary was loaded** — this is why local debugging "just works" for private builds.
2. **The hard-coded build path** embedded in the PE Debug Directories (e.g., `C:\build\obj\Debug\MyApp.pdb`).
3. **The Symbol Server cache directory** — a local cache of PDBs previously downloaded from the Symbol Server.
4. **The Symbol Server itself** — the remote store where public build PDBs are archived.

This search order is why private and public builds never conflict: your local build's PDB sits right next to the binary and is found first.

```ad-note
You can observe this search process in Visual Studio by opening **Debug > Windows > Modules** and checking the **Symbol File** column and **Symbol Status** while debugging.
```

---

## DUMPBIN — Inspecting Debug Directories

You can verify the GUID embedded in a binary using `DUMPBIN`, a command-line tool included with Visual Studio. Run it from a **Developer Command Prompt**:

```powershell
dumpbin /HEADERS MyApp.dll
```

The relevant output is the **Debug Directories** section:

```
Debug Directories

  Time  Type  Size  RVA      Pointer
  ----  ----  ----  -------  -------
  ...   cv    4A    000025C4 7C4     Format: RSDS,
    {4B46C704-B6DE-44B2-B8F5-A200A7E541B0}, 1,
    C:\build\obj\Debug\MyApp.pdb
```

Key fields:

- **GUID** (`{4B46C704-...}`) — must match the PDB's internal GUID exactly.
- **Build path** (`C:\build\obj\Debug\MyApp.pdb`) — the second location the debugger searches.
- **Format: RSDS** — indicates the modern PDB format (as opposed to the older NB10 format).

```ad-info
For .NET projects, MSBuild actually compiles to the `obj\<Config>` directory first and only copies to the output (`bin\Debug` or `bin\Release`) on a successful build. This is why the embedded path often points to `obj\` rather than `bin\`.
```

---

## Symbol Server

A **Symbol Server** is a structured store (file share, HTTP server, or cloud service) that archives PDB files and binaries for all public builds. It solves the fundamental problem: when someone reports a crash from a production build shipped three months ago, you need the exact PDB for that build.

Why every team needs one:

- **Automatic matching** — Visual Studio and WinDBG query the Symbol Server by GUID automatically. No manual file hunting.
- **Historical coverage** — every public build's symbols are preserved indefinitely.
- **Team-wide access** — any developer can debug any build without asking "who has the PDB?"

Setting up a Symbol Server in Visual Studio:

1. **Tools > Options > Debugging > Symbols**
2. Add your server URL or UNC path (e.g., `\\buildserver\symbols` or `https://symbols.mycompany.com`)
3. Set a local cache directory for downloaded PDBs

```ad-note
Microsoft publishes public symbols for the .NET runtime and Windows OS components via their public Symbol Server at `https://msdl.microsoft.com/download/symbols`. Adding this to your debugger configuration lets you step into framework code during debugging.
```

### Publishing Symbols

Use the `symstore.exe` tool (part of Debugging Tools for Windows) or the `dotnet symbol` tool to add PDBs to your Symbol Server as part of your CI/CD pipeline:

```powershell
symstore add /r /f "C:\build\output\*.pdb" /s "\\buildserver\symbols" /t "MyApp" /v "2.1.0"
```

---

## Source Server and Source Indexing

Even with the right PDB, you still need the **exact source files** that were compiled into the build. Source indexing solves this by embedding version control commands directly into the PDB file.

How it works:

1. After compilation, the **source indexing tool** (`srctool.exe`, `sourcelink`, etc.) scans the PDB's source file references.
2. It replaces the local file paths with **version control retrieval commands** (e.g., `git checkout <commit-sha> -- path/to/file.cs`).
3. When the debugger needs a source file, it executes the embedded command to pull the exact revision from version control into a local cache.

**Modern .NET alternative — SourceLink:**

SourceLink is the modern replacement for traditional source indexing. It embeds a URL pattern in the PDB that maps source files to a web-accessible repository (GitHub, Azure DevOps, etc.). The debugger fetches source files on demand via HTTPS.

```xml
<!-- Enable SourceLink in .csproj -->
<PropertyGroup>
    <PublishRepositoryUrl>true</PublishRepositoryUrl>
    <EmbedUntrackedSources>true</EmbedUntrackedSources>
</PropertyGroup>

<ItemGroup>
    <!-- For GitHub-hosted repos -->
    <PackageReference Include="Microsoft.SourceLink.GitHub" Version="8.0.0" PrivateAssets="All" />
</ItemGroup>
```

```ad-info
SourceLink works with GitHub, Azure DevOps, GitLab, Bitbucket, and other providers. NuGet packages published with SourceLink-enabled PDBs allow consumers to step into library source code during debugging — a significant improvement for the .NET ecosystem.
```

---

## GAC and DEVPATH

Debugging assemblies in the **Global Assembly Cache (GAC)** presents a unique challenge. The GAC stores assemblies in a structured directory hierarchy under `C:\Windows\assembly\` (or `C:\Windows\Microsoft.NET\assembly\` in .NET 4+), and the physical path is not straightforward:

```
C:\Windows\assembly\GAC_MSIL\Example\1.0.0.0__682bc775ff82796a\Example.dll
```

For **private builds**, the debugger finds the PDB via the embedded build path — no problem. But on a **different machine** (e.g., a test server), you would need to manually copy the PDB into the GAC's internal directory structure, which is unsupported and fragile.

### The DEVPATH Solution

**DEVPATH** is a .NET Framework mechanism that adds a developer-specified directory to the assembly search path, effectively extending the GAC without the GAC's versioning enforcement.

Setup steps:

1. **Create a directory** with read access for all accounts and write access for your dev account (e.g., `C:\DevAssemblies`).
2. **Set a system environment variable** named `DEVPATH` with the directory path as its value. Set this *before* the next step.
3. **Add the following to your configuration file** (`App.config`, `Web.config`, or `Machine.config`):

```xml
<configuration>
    <runtime>
        <developmentMode developerInstallation="true" />
    </runtime>
</configuration>
```

Now you can drop an assembly and its PDB into the DEVPATH directory. The runtime will find the assembly there, and the debugger will find the PDB alongside it.

```ad-warning
**Use DEVPATH only on development machines.** Assemblies in the DEVPATH directory are **not version-checked** — the runtime loads whatever it finds, regardless of version. If you accidentally enable this in `Machine.config`, every .NET application on the machine is affected. A misconfigured DEVPATH (missing environment variable or nonexistent path) causes applications to crash at startup with a `COMException` and the misleading message: *"Invalid value for registry."*
```

```ad-note
DEVPATH is a **.NET Framework** feature. In modern .NET (Core/.NET 5+), the GAC does not exist and assemblies are resolved through the deps file and runtime probing — making DEVPATH unnecessary. If you are working with .NET Framework and GAC assemblies (e.g., SharePoint web parts), DEVPATH remains the cleanest debugging approach.
```

---

## Source File Path Storage

Understanding how source paths are stored in PDB files helps you set up debuggable builds on machines other than the original build machine.

**For source-indexed public builds:** The PDB contains version control retrieval commands (or SourceLink URLs) instead of local paths. The debugger fetches source automatically — no path matching needed.

**For non-indexed builds (including all private builds):** The PDB stores the **full absolute path** to each source file as it existed on the build machine (e.g., `C:\src\MyProject\Program.cs`). The debugger will look for the source at that exact path.

This creates a problem when debugging a build on a different machine — the original path likely does not exist.

### The SUBST.EXE Trick

`SUBST` maps a directory path to a virtual drive letter, giving you control over the paths embedded in PDB files:

**On the build machine:**

```powershell
# Map the source root to a consistent drive letter
subst R: C:\BuildAgent\src

# Build from the mapped drive — PDB now contains R:\MyProject\Program.cs
cd R:\MyProject
dotnet build
```

**On any debugging machine:**

```powershell
# Pull source to any location
git clone https://repo.example.com/MyProject C:\Users\dev\source\MyProject

# Map it to the same drive letter the build used
subst R: C:\Users\dev\source\MyProject

# The debugger now finds source files at R:\MyProject\*.cs — matching the PDB
```

This approach is preferable to configuring source search directories in the debugger (which can be unreliable with complex project structures). Set the `SUBST` command as a startup item on your build machine so it runs automatically when the build agent logs in.

```ad-note
`SUBST` avoids the need for network shares (which some organizations disable via `DRIVE$` restrictions). The mapped drive appears in Explorer like a real drive and works with all tools — compilers, debuggers, scripts — transparently.
```

---

## See Also

- [[The Role of .NET Assemblies]]
- [[Format of a .NET Assembly]]
- [[Global Assembly Cache]]
