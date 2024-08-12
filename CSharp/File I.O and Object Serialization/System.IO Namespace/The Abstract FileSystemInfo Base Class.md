- Provides the base class for both FileInfo and DirectoryInfo objects.

![[Pasted image 20240711143202.png|center|700]]

```ad-note
FileSystemInfo also defines the Delete() method. This is implemented by derived types to delete a given file or directory from the hard drive. Also, you can call Refresh() prior to obtaining attribute information to ensure that the statistics regarding the current file (or directory) are not outdated.
```