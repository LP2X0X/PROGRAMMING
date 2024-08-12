Signing an assembly involves applying a strong name to it, which includes a public key and a digital signature. This process ensures the integrity and uniqueness of the assembly, making it possible to store it in the Global Assembly Cache (GAC) and to avoid name conflicts.

### Steps to Sign an Assembly

1. **Generate a Strong Name Key File**:
   - Use the `sn.exe` (Strong Name) tool to generate a strong name key pair.

   ```sh
   sn -k MyKey.snk
   ```

2. **Add the Key File to the Assembly**:
   - Add an `AssemblyKeyFile` attribute to your AssemblyInfo.cs file or include it in your project file to specify the key file to use for signing.

3. **Compile the Assembly with the Strong Name**:
   - Compile the assembly with the strong name key.

### Detailed Example

#### Step 1: Generate a Strong Name Key File

Open a command prompt and run the following command:

```sh
sn -k MyKey.snk
```

This command generates a key pair and saves it in `MyKey.snk`.

#### Step 2: Add the Key File to the Assembly

Add the `AssemblyKeyFile` attribute to your `AssemblyInfo.cs` file:

```csharp
using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

// General Information about an assembly is controlled through the following 
// set of attributes. Change these attribute values to modify the information
// associated with an assembly.
[assembly: AssemblyTitle("MyAssembly")]
[assembly: AssemblyDescription("")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("")]
[assembly: AssemblyProduct("MyAssembly")]
[assembly: AssemblyCopyright("Copyright ©  2023")]
[assembly: AssemblyTrademark("")]
[assembly: AssemblyCulture("")]

// Setting ComVisible to false makes the types in this assembly not visible 
// to COM components.  If you need to access a type in this assembly from 
// COM, set the ComVisible attribute to true on that type.
[assembly: ComVisible(false)]

// The following GUID is for the ID of the typelib if this project is exposed to COM
[assembly: Guid("d69d6a4c-6b2b-4d7e-8f1a-9dd233b987dd")]

// Version information for an assembly consists of the following four values:
//
//      Major Version
//      Minor Version 
//      Build Number
//      Revision
//
// You can specify all the values or you can default the Build and Revision Numbers 
// by using the '*' as shown below:
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]

// Add the AssemblyKeyFile attribute
[assembly: AssemblyKeyFile("MyKey.snk")]
```

Alternatively, you can specify the key file directly in your project file (`.csproj`):

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Library</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <AssemblyOriginatorKeyFile>MyKey.snk</AssemblyOriginatorKeyFile>
    <SignAssembly>true</SignAssembly>
  </PropertyGroup>
</Project>
```

#### Step 3: Compile the Assembly with the Strong Name

Build your project as usual, and the assembly will be signed with the strong name specified in the key file.

### Verifying the Strong Name

To verify that the assembly has been correctly signed, you can use the `sn.exe` tool:

```sh
sn -v MyAssembly.dll
```

This command verifies the strong name signature of the specified assembly.

### Using Signed Assemblies

Once your assembly is signed, you can install it into the GAC using the `gacutil` tool:

```sh
gacutil /i MyAssembly.dll
```

### Example Project Structure

Here’s an example structure of your project:

```
MyAssembly/
├── MyKey.snk
├── MyAssembly.csproj
└── AssemblyInfo.cs
```

### Summary

By following these steps, you can generate a strong name key, sign your assembly with this key, and verify the signature. Signed assemblies ensure integrity and uniqueness, allowing them to be shared across multiple applications and stored in the GAC.