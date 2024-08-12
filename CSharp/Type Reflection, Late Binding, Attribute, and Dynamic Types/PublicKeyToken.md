The PublicKeyToken is a crucial part of an assembly's strong name, which uniquely identifies the assembly. It is a short, 8-byte hash derived from the public key used to sign the assembly. The PublicKeyToken is used to uniquely identify and verify the authenticity of the assembly, especially when dealing with shared libraries stored in the Global Assembly Cache (GAC).

### Retrieving the PublicKeyToken of an Assembly

To retrieve the PublicKeyToken of an assembly programmatically, you can use the `AssemblyName` class and its properties. Here’s how you can do it:

### Example Code

```csharp
using System;
using System.Reflection;
using System.Text;

class Program
{
    static void Main(string[] args)
    {
        // Path to the assembly
        string assemblyPath = "MyLibrary.dll"; // Change to your assembly path
        
        // Load the assembly
        Assembly assembly = Assembly.LoadFrom(assemblyPath);
        
        // Get the AssemblyName
        AssemblyName assemblyName = assembly.GetName();
        
        // Display the Full Name (including the PublicKeyToken)
        Console.WriteLine("Full Name: " + assemblyName.FullName);
        
        // Get the PublicKeyToken as a byte array
        byte[] publicKeyToken = assemblyName.GetPublicKeyToken();
        
        // Convert the byte array to a hex string
        string publicKeyTokenString = BitConverter.ToString(publicKeyToken).Replace("-", "").ToLowerInvariant();
        
        // Display the PublicKeyToken
        Console.WriteLine("PublicKeyToken: " + publicKeyTokenString);
    }
}
```

### Explanation

1. **Loading the Assembly**:
   - The `Assembly.LoadFrom` method is used to load the assembly from the specified path.
   
2. **Getting the AssemblyName**:
   - The `GetName` method retrieves the `AssemblyName` object for the loaded assembly.

3. **Displaying the Full Name**:
   - The `FullName` property of `AssemblyName` includes the assembly's name, version, culture, and PublicKeyToken.

4. **Retrieving the PublicKeyToken**:
   - The `GetPublicKeyToken` method returns the PublicKeyToken as a byte array.

5. **Converting to Hex String**:
   - The `BitConverter.ToString` method converts the byte array to a string of hex values. The `Replace("-", "")` method removes the hyphens, and `ToLowerInvariant` ensures the string is in lowercase.

### Output Example

```
Full Name: MyLibrary, Version=1.0.0.0, Culture=neutral, PublicKeyToken=abcdef1234567890
PublicKeyToken: abcdef1234567890
```

This example demonstrates how to load an assembly, retrieve its `AssemblyName`, and extract the PublicKeyToken. The PublicKeyToken is displayed both as part of the full assembly name and separately as a hexadecimal string.

### Retrieving PublicKeyToken from the GAC

If you want to list all assemblies in the GAC and retrieve their PublicKeyTokens, you can extend the previous example to enumerate the assemblies in the GAC:

### Example Code for Enumerating GAC Assemblies

```csharp
using System;
using System.Collections.Generic;
using System.Reflection;
using System.Runtime.InteropServices;
using System.Text;

class Program
{
    static void Main(string[] args)
    {
        List<AssemblyName> gacAssemblies = GetGacAssemblies();
        foreach (AssemblyName assemblyName in gacAssemblies)
        {
            // Display the Full Name (including the PublicKeyToken)
            Console.WriteLine("Full Name: " + assemblyName.FullName);
            
            // Get the PublicKeyToken as a byte array
            byte[] publicKeyToken = assemblyName.GetPublicKeyToken();
            
            // Convert the byte array to a hex string
            string publicKeyTokenString = BitConverter.ToString(publicKeyToken).Replace("-", "").ToLowerInvariant();
            
            // Display the PublicKeyToken
            Console.WriteLine("PublicKeyToken: " + publicKeyTokenString);
            Console.WriteLine();
        }
    }

    static List<AssemblyName> GetGacAssemblies()
    {
        List<AssemblyName> assemblies = new List<AssemblyName>();
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
            assemblies.Add(new AssemblyName(name.ToString()));
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
```

This example builds on the previous one by listing all assemblies in the GAC and displaying their full names and PublicKeyTokens. This approach ensures that you can dynamically inspect and retrieve detailed information about assemblies stored in the GAC.