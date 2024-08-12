The Global Assembly Cache (GAC) is a machine-wide repository used to store assemblies that are shared by multiple applications on a system. Assemblies in the GAC are available to all applications running on the system without needing to be located in the application's directory.

### Working with the Global Assembly Cache

1. **Installing an Assembly into the GAC**:
   - You can use the `gacutil` tool to install an assembly into the GAC. This tool is typically available with the .NET SDK.

   ```sh
   gacutil /i MyLibrary.dll
   ```

2. **Listing Assemblies in the GAC**:
   - You can use the `gacutil` tool to list all assemblies installed in the GAC.

   ```sh
   gacutil /l
   ```

3. **Uninstalling an Assembly from the GAC**:
   - You can use the `gacutil` tool to uninstall an assembly from the GAC.

   ```sh
   gacutil /u MyLibrary
   ```

4. **Programmatically Accessing the GAC**:
   - You can programmatically access assemblies in the GAC using reflection.

### Example: Displaying Assembly Names from the GAC

Here's how you can list the names of assemblies installed in the GAC using C#:

```csharp
using System;
using System.Reflection;
using System.Collections.Generic;
using System.Runtime.InteropServices;

namespace GacExample
{
    class Program
    {
        static void Main(string[] args)
        {
            List<string> assemblies = GetGacAssemblies();
            foreach (string assemblyName in assemblies)
            {
                Console.WriteLine(assemblyName);
            }
        }

        static List<string> GetGacAssemblies()
        {
            List<string> assemblies = new List<string>();
            IAssemblyEnum assemblyEnum = null;
            IApplicationContext applicationContext = null;
            IAssemblyName assemblyName = null;

            Fusion.CreateAssemblyEnum(out assemblyEnum, null, null, 2, 0);

            while (assemblyEnum.GetNextAssembly(out applicationContext, out assemblyName, 0) == 0)
            {
                uint nameLength = 0;
                assemblyName.GetDisplayName(null, ref nameLength, 0);
                StringBuilder name = new StringBuilder((int)nameLength);
                assemblyName.GetDisplayName(name, ref nameLength, 0);
                assemblies.Add(name.ToString());
            }

            return assemblies;
        }

        // P/Invoke declarations for fusion.dll
        private static class Fusion
        {
            [DllImport("fusion.dll")]
            public static extern int CreateAssemblyEnum(out IAssemblyEnum ppEnum, IntPtr pUnkReserved, IAssemblyName pName, uint dwFlags, IntPtr pvReserved);
        }

        [ComImport, InterfaceType(ComInterfaceType.InterfaceIsIUnknown), Guid("21b8916c-f28e-11d2-a473-00c04f8ef448")]
        private interface IAssemblyEnum
        {
            [PreserveSig]
            int GetNextAssembly(out IApplicationContext ppAppCtx, out IAssemblyName ppName, uint dwFlags);
        }

        [ComImport, InterfaceType(ComInterfaceType.InterfaceIsIUnknown), Guid("7c23ff90-33af-11d3-95da-00a024a85b51")]
        private interface IApplicationContext { }

        [ComImport, InterfaceType(ComInterfaceType.InterfaceIsIUnknown), Guid("8c23b657-4b1b-11d3-81b1-00a0c9231b9c")]
        private interface IAssemblyName
        {
            [PreserveSig]
            int GetDisplayName(StringBuilder szDisplayName, ref uint pccBuffer, uint dwDisplayFlags);
        }
    }
}
```

### Explanation

- **P/Invoke**: The code uses Platform Invocation Services (P/Invoke) to call unmanaged functions from `fusion.dll`, which provides access to the GAC.
- **`CreateAssemblyEnum`**: This function creates an enumerator that can iterate through the assemblies in the GAC.
- **`GetNextAssembly`**: This function gets the next assembly in the enumeration.
- **`GetDisplayName`**: This function gets the display name of the assembly.

### Steps to Run the Example

1. **Compile the Code**: Save the code to a file, e.g., `GacExample.cs`, and compile it.

   ```sh
   csc GacExample.cs
   ```

2. **Run the Compiled Program**:

   ```sh
   GacExample.exe
   ```

The program will list the names of the assemblies installed in the GAC. This example demonstrates how to programmatically access the GAC and retrieve information about the assemblies stored there.