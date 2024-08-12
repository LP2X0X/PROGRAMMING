To generate an IL (Intermediate Language) file that includes metadata tokens, you can use the IL Disassembler tool (ILDASM) provided by the .NET SDK. ILDASM can be used to disassemble a .NET assembly (DLL or EXE) and output the IL code along with the metadata tokens.

Here are the steps to generate the IL file:

1. **Build Your Project**:
   - Ensure your C# project is built and you have the compiled assembly (DLL or EXE file).

2. **Open ILDASM**:
   - ILDASM is included with the .NET SDK and can be found in the `C:\Program Files\dotnet\sdk\<version>\` directory.
   - Alternatively, you can open it from the Developer Command Prompt by typing `ildasm`.

3. **Load Your Assembly**:
   - In ILDASM, go to `File` > `Open` and select your compiled assembly (DLL or EXE file).

4. **Generate the IL File**:
   - Once the assembly is loaded in ILDASM, go to `File` > `Dump` or `File` > `Dump` with options to specify additional options for the dump.
   - In the "Dump Options" dialog, you can select various options, such as including header information, tokens, and more.
   - Choose the options that suit your needs and click `OK`.
   - Save the output to a `.il` file.

Here is an example of what you might see in the generated IL file, including tokens:

```il
// Metadata version: v4.0.30319
.assembly extern mscorlib
{
  .publickeytoken = (B7 7A 5C 56 19 34 E0 89 )
  .ver 4:0:0:0
}
.assembly MyAssembly
{
  .hash algorithm 0x00008004
  .ver 1:0:0:0
}
.module MyAssembly.dll
// MVID: {F4A83B8A-9F64-4F7A-97D4-4A63426E41F8}
.imagebase 0x00400000
.file alignment 0x00000200
.stackreserve 0x00100000
.subsystem 0x0003       // WINDOWS_CUI
.corflags 0x00000001    // ILONLY

// =============== CLASS MEMBERS DECLARATION ===================

.class public auto ansi beforefieldinit MyNamespace.MyClass
       extends [mscorlib]System.Object
{
  .field public int32 MyField

  .method public hidebysig instance void 
          MyMethod() cil managed
  {
    // Code size       2 (0x2)
    .maxstack  8
    IL_0000:  nop
    IL_0001:  ret
  } // end of method MyClass::MyMethod

  .method public hidebysig specialname rtspecialname 
          instance void  .ctor() cil managed
  {
    // Code size       8 (0x8)
    .maxstack  8
    IL_0000:  ldarg.0
    IL_0001:  call       instance void [mscorlib]System.Object::.ctor()
    IL_0006:  nop
    IL_0007:  ret
  } // end of method MyClass::.ctor

} // end of class MyNamespace.MyClass
```

In this example, the `.field` and `.method` definitions will include tokens, which are hexadecimal identifiers prefixed with `0x`. These tokens correspond to metadata entries for the type, field, and methods.

### Automating with Command Line

You can also use ILDASM from the command line to automate this process. Here is a command to disassemble an assembly and include metadata tokens:

```sh
ildasm MyAssembly.dll /out=MyAssembly.il /tokens
```

This command disassembles `MyAssembly.dll` and outputs the IL code to `MyAssembly.il`, including metadata tokens.

### Viewing IL Code

You can open the generated `.il` file with any text editor to view the IL code along with the metadata tokens. This file will include detailed information about the structure and contents of your assembly, which is useful for debugging and analyzing .NET assemblies at a low level.