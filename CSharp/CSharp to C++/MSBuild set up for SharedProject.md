### **“How can I make my C++ project's DLL always get copied (or updated) to a Shared Project that is referenced by a C# app, every time I rebuild the C++ project?”**

This is a **build-time synchronization** task — and it's absolutely doable using **MSBuild automation**.

---

## ✅ Goal
When you **build your native C++ DLL project**, the output `.dll` file is **automatically copied** to a folder inside your **Shared Project** (e.g. `NativeDll.SharedProject\runtimes\win-x64\native\`) so it gets included and loaded by the C# app.

---

## ✅ Approach  
We will edit the **C++ project’s .vcxproj** file to include a **post-build event**, or even better, an **MSBuild Target**, to copy the DLL to the shared project folder.

---

## 🛠 Method 1: Add Post-Build Event to Native C++ Project

### 🔹 Steps

1. Right-click on your C++ `.vcxproj` → **Properties**
2. Go to:
   - **Configuration Properties** → **Build Events** → **Post-Build Event**
3. Use a command like this (adjust paths as needed):

```bash
xcopy /Y /D "$(TargetPath)" "$(SolutionDir)NativeDll.SharedProject\runtimes\win-x64\native\"
```

### 💡 Explanation:
- `$(TargetPath)` → full path of the built DLL
- `/Y` → auto-approve overwrite
- `/D` → skip copying if file hasn't changed
- `$(SolutionDir)` → root of the solution
- Adjust output path for architecture (`win-x64` vs `win-x86`)

---

## ✅ Method 2: Use MSBuild Target (Recommended for Flexibility)

Edit your `.vcxproj` file manually and add this to the bottom _before_ the closing `</Project>` tag:

```xml
<Target Name="CopyOutputToSharedProject" AfterTargets="Build">
  <ItemGroup>
    <SharedProjectNative Include="$(TargetPath)" />
  </ItemGroup>
  <Copy
    SourceFiles="@(SharedProjectNative)"
    DestinationFolder="$(SolutionDir)NativeDll.SharedProject\runtimes\win-x64\native\"
    SkipUnchangedFiles="true"
    OverwriteReadOnlyFiles="true" />
</Target>
```

### 🔍 Benefits:
- MSBuild-native (cross-platform toolchain support)
- Executes automatically after build
- No need to use fragile `xcopy` or hardcoded paths

---

## ✅ Bonus: Add `.targets` file in Shared Project for Output Copying

In `NativeDll.SharedProject`, create a file named `NativeDll.SharedProject.targets`, and include:

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>
    <None Include="runtimes\win-x64\native\NativeDll.dll">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
</Project>
```

✅ This tells the C# project consuming the shared project:
- → _“Please copy `NativeDll.dll` to the app output when building.”_

---

## 🔄 Resulting Build Workflow

When you:
- 🧱 Build the C++ DLL
   - The DLL is automatically copied to your Shared Project output path
- ⚙ Build or run the C# project
   - The Shared Project ensures that DLL is copied to the final output (e.g., `bin/Debug`)

✔️ End result: Native DLL is always synced and loaded — even with Just-In-Time changes or rebuilds.

---

## ✅ Summary

| Task | Tool |
|------|------|
| Auto-copy DLL to Shared Project | Post-Build Event or MSBuild Target in `.vcxproj` |
| Ensure DLL reaches final EXE output | `.targets` file in Shared Project |
| Architecture-aware paths | Put DLL in `runtimes\win-x64\native\` |
| Avoid manual copying     | ✅ Fully automated build process |
