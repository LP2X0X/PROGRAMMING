- The operating system file header establishes the fact that the assembly can be loaded and manipulated by the target operating system (in the following example, Windows). This header data also identifies the kind of application (console-based, GUI-based, or \*.dll code library) to be hosted by the operating system.
- Open the CarLibrary.dll file (in the book’s repo or created later in this chapter) using the dumpbin.exe utility (via the developer command prompt) with the /headers flag as so:  

```ad-note
dumpbin /headers CarLibrary.dll
```