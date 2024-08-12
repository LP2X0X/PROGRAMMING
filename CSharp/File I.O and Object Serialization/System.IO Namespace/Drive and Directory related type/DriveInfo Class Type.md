The `DriveInfo` class in the `System.IO` namespace provides information about the drives on the current system. This includes information about available free space, total size, drive type, drive format, and more.

### Common Properties:

- `AvailableFreeSpace`: Gets the amount of available free space on the drive.
- `DriveFormat`: Gets the name of the file system, such as NTFS or FAT32.
- `DriveType`: Gets the type of the drive (e.g., `Fixed`, `Removable`, `Network`).
- `IsReady`: Gets a value indicating whether the drive is ready.
- `Name`: Gets the name of the drive.
- `RootDirectory`: Gets the root directory of the drive.
- `TotalFreeSpace`: Gets the total amount of free space available on the drive.
- `TotalSize`: Gets the total size of storage space on the drive.
- `VolumeLabel`: Gets or sets the volume label of the drive.

### Common Methods:

- `GetDrives()`: Retrieves the drive names of all logical drives on a computer.

### Example:

Here is an example demonstrating how to use the `DriveInfo` class to get information about all drives on the system:

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        // Get all drives on the system
        DriveInfo[] allDrives = DriveInfo.GetDrives();

        foreach (DriveInfo drive in allDrives)
        {
            Console.WriteLine($"Drive: {drive.Name}");
            Console.WriteLine($"Drive Type: {drive.DriveType}");

            if (drive.IsReady)
            {
                Console.WriteLine($"Volume Label: {drive.VolumeLabel}");
                Console.WriteLine($"File System: {drive.DriveFormat}");
                Console.WriteLine($"Available Free Space: {drive.AvailableFreeSpace} bytes");
                Console.WriteLine($"Total Free Space: {drive.TotalFreeSpace} bytes");
                Console.WriteLine($"Total Size: {drive.TotalSize} bytes");
            }

            Console.WriteLine();
        }
    }
}
```

### Explanation:

1. **Get Drives**:
    - The `DriveInfo.GetDrives()` method retrieves an array of `DriveInfo` objects, one for each drive on the current system.

2. **Iterate Through Drives**:
    - The program iterates through each `DriveInfo` object in the array.

3. **Drive Information**:
    - For each drive, it prints the name and type.
    - If the drive is ready (`drive.IsReady` is `true`), it also prints the volume label, file system, available free space, total free space, and total size.

### Drive Types:

The `DriveType` property can return the following types (among others):
- `DriveType.CDRom`: The drive is an optical disc device, such as a CD or DVD-ROM.
- `DriveType.Fixed`: The drive is a fixed disk.
- `DriveType.Network`: The drive is a network drive.
- `DriveType.NoRootDirectory`: The drive does not have a root directory.
- `DriveType.Ram`: The drive is a RAM disk.
- `DriveType.Removable`: The drive is a removable storage device, such as a USB flash drive.
- `DriveType.Unknown`: The drive type is unknown.

The `DriveInfo` class provides a simple and efficient way to gather detailed information about the system's storage devices.