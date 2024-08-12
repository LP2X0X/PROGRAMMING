C# allows you to define a static constructor, which allows you to safely set the values of your static data.
The CoreCLR calls all static constructors before the first use (and never calls them again for that instance of the application).
Here are a few points of interest regarding static constructors:  
• A given class may define only a single static constructor. In other words, the static constructor cannot be overloaded.  
• A static constructor does not take an access modifier and cannot take any parameters.  
• A static constructor executes exactly one time, regardless of how many objects of the type are created.  
• The runtime invokes the static constructor when it creates an instance of the class or before accessing the first static member invoked by the caller.  
• The static constructor executes before any instance-level constructors.