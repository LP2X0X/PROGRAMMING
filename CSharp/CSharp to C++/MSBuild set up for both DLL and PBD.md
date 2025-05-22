### ✅ Automatically copy _**both** the `.dll` **and** the `.pdb` file_ from your **C++ native project** into your **Shared Project**'s native runtime folder when you rebuild the C++ project.

This is a common practice, especially to enable debugging of native C++ code from managed C# (via symbols in `.pdb`).

---

## 💡 Solution:
Modify your **C++ `.vcxproj`** to copy both files during build.

### ✅ Option 1: Post-Build Event (Quick & Visual Studio Friendly)

Go to:

**C++ project → Properties → Build Events → Post-Build Event**

Paste this:

```bash
xcopy /Y /D "$(TargetPath)" "$(SolutionDir)NativeDll.SharedProject\runtimes\win-x64\native\"
xcopy /Y /D "$(TargetDir)$(TargetName).pdb" "$(SolutionDir)NativeDll.SharedProject\runtimes\win-x64\native\"
```

### 📌 Paths:
- `$(TargetPath)`: Full path to `NativeDll.dll`
- `$(TargetDir)`: Folder where outputs go
- `$(TargetName)`: The name, without extension → `NativeDll`
- `$(TargetName).pdb`: The symbols file

---

### ✅ Option 2: MSBuild `<Target>` – Preferred for Flexibility

Instead of post-build script, edit the `.vcxproj` file and paste this before the closing `</Project>` tag:

```xml
<Target Name="CopyOutputToSharedProject" AfterTargets="Build">
  <ItemGroup>
    <FilesToCopy Include="$(TargetPath)" />
    <FilesToCopy Include="$(TargetDir)$(TargetName).pdb" />
  </ItemGroup>

  <Copy
    SourceFiles="@(FilesToCopy)"
    DestinationFolder="$(SolutionDir)NativeDll.SharedProject\runtimes\win-x64\native\"
    SkipUnchangedFiles="true"
    OverwriteReadOnlyFiles="true" />
</Target>
```

---

## ✅ Bonus: Ensure C# Project Copies DLL and PDB

In your **Shared Project**, add/edit:

📄 `NativeDll.SharedProject.targets`

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>
    <None Include="runtimes\win-x64\native\NativeDll.dll">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
    <None Include="runtimes\win-x64\native\NativeDll.pdb">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
</Project>
```

🔁 This ensures both **`.dll` and `.pdb`** make it into your **C# app’s output folder**, and symbol loading will work for mixed C++/C# debugging.

---

## ✅ Summary

| Task                          | ✅ Solution                              |
|-------------------------------|------------------------------------------|
| Copy DLL after C++ build      | MSBuild `<Target>`, or Post-Build Script |
| Copy PDB after build          | Add it to the same operation             |
| Keep C# output synced         | Add `.targets` to Shared Project         |
| Enable native debugging       | `.pdb` presence required in output       |
