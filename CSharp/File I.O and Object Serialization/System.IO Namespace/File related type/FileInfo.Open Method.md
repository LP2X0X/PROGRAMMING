- You can use the FileInfo.Open() method to open existing files, as well as to create new files with far more precision than you can with FileInfo.Create(). This works because Open() typically takes several parameters to qualify exactly how to iterate the file you want to manipulate. Once the call to Open() completes, you are returned a FileStream object.
- This version of the overloaded Open() method requires three parameters:
	- The first parameter of the Open() method specifies the general flavor of the I/O request (e.g., make a new file, open an existing file, and append to a file) ![[Pasted image 20240723090511.png|center|700]]
	- You use the second parameter of the Open() method, a value from the FileAccess enumeration, to determine the read/write behavior of the underlying stream ![[Pasted image 20240723090832.png|center|300]]
	- The third and final parameter specifies how to share the file among other file handlers ![[Pasted image 20240723091024.png|center|300]]

---
- References: https://learn.microsoft.com/en-us/dotnet/api/system.io.fileshare?view=net-8.0