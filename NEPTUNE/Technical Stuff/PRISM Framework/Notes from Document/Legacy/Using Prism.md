# Summary
![[Pasted image 20240118103230.png|center]]
# Step 1: Define the Shell
- [[BootstrapperShell|Shell usually is a Window control]].
- Shells typically set the appearance for the entire application and contain the styles that are used throughout the application.
![[Ch1IntroFig5.png|center]]
# Step 2: Create the Bootstrapper
- The bootstrapper is the glue that connects the application with the Prism Library services and the Unity or MEF containers.
![[Ch1IntroFig6.png|center]]
# Step 3: Create the module
- A module is denoted by a class that implements the IModule interface. These modules, during initialization, register their views and services and may add one or more views to the shell. Depending on your module discovery approach, you may need to apply attributes to your module classes or define dependencies between your modules.