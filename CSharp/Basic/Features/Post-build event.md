- This code snippet help copy the dll from a project output directory to the application directory (which is the startup project directory):

```csharp
  <PropertyGroup>
    <PostBuildEvent>
		xcopy /Y "$(TargetDir)\<NAME OF DLL>.dll" "$(SolutionDir)<NAME OF START UP PROJECT>\bin\$(Configuration)"
		xcopy /Y "$(TargetDir)<NAME OF DLL>.dll" "$(SolutionDir)<NAME OF START UP PROJECT>\bin\$(Configuration)"
    </PostBuildEvent>
  </PropertyGroup>
```