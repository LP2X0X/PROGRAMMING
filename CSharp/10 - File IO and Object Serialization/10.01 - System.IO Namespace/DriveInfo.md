---
tags:
 - csharp
 - io
 - drives
---

## DriveInfo

`DriveInfo` is an instance class that provides information about a drive on the system -- its capacity, free space, file system format, volume label, and type. It is primarily used for diagnostics, disk space monitoring, and deciding where to write files.

Unlike `FileInfo` and `DirectoryInfo`, there is no static counterpart. You either create a `DriveInfo` for a specific drive or enumerate all drives with `DriveInfo.GetDrives()`.

---

## Getting Drive Information

### Enumerate All Drives

```csharp
DriveInfo[] drives = DriveInfo.GetDrives();

foreach (DriveInfo drive in drives)
{
    Console.WriteLine($"Drive: {drive.Name}");  // "C:\", "D:\", etc.
}
```

### Create for a Specific Drive

```csharp
// Pass the drive letter (with or without trailing backslash)
DriveInfo cDrive = new DriveInfo("C");
DriveInfo dDrive = new DriveInfo("D:\\");

Console.WriteLine(cDrive.TotalSize);       // total capacity in bytes
Console.WriteLine(cDrive.AvailableFreeSpace); // free space in bytes
```

---

## Properties

| Property | Type | Description |
|---|---|---|
| `Name` | `string` | Drive name (e.g., `"C:\"`) |
| `DriveType` | `DriveType` | Type of drive (Fixed, Removable, Network, etc.) |
| `VolumeLabel` | `string` | Volume label (e.g., `"Windows"`, `"Data"`) -- get or set |
| `DriveFormat` | `string` | File system name (e.g., `"NTFS"`, `"FAT32"`, `"exFAT"`, `"ext4"`) |
| `TotalSize` | `long` | Total drive capacity in bytes |
| `TotalFreeSpace` | `long` | Total free space in bytes (includes reserved space) |
| `AvailableFreeSpace` | `long` | Free space available to the current user (respects disk quotas) |
| `IsReady` | `bool` | Whether the drive is ready (media is present and accessible) |
| `RootDirectory` | `DirectoryInfo` | Root directory of the drive as a `DirectoryInfo` object |

```ad-warning
title: Always Check IsReady Before Accessing Properties
Accessing `TotalSize`, `AvailableFreeSpace`, `VolumeLabel`, or `DriveFormat` on a drive that is not ready (e.g., an empty CD-ROM drive, an ejected USB stick) throws an `IOException`. Always check `IsReady` first.
```

### TotalFreeSpace vs AvailableFreeSpace

These two properties are often confused:

| Property | What It Measures |
|---|---|
| `TotalFreeSpace` | Total free bytes on the drive, ignoring disk quotas |
| `AvailableFreeSpace` | Free bytes available to the **current user**, respecting disk quotas |

On systems without disk quotas (most personal machines), these values are identical. On enterprise servers with per-user quotas, `AvailableFreeSpace` may be significantly less than `TotalFreeSpace`.

```ad-note
title: Which Free Space Property to Use
Use `AvailableFreeSpace` for "can the current user write X bytes?" checks. Use `TotalFreeSpace` for overall drive health monitoring. When in doubt, prefer `AvailableFreeSpace` -- it gives you the conservative (safe) answer.
```

---

## DriveType Enum

| Value | Description | Examples |
|---|---|---|
| `Fixed` | Non-removable internal drive | SSD, HDD, NVMe |
| `Removable` | Removable storage | USB flash drive, SD card |
| `Network` | Mapped network drive | `\\server\share` mapped to `Z:\` |
| `CDRom` | Optical disc drive | CD, DVD, Blu-ray |
| `Ram` | RAM disk | Virtual drives backed by memory |
| `Unknown` | Drive type cannot be determined | Rare edge cases |
| `NoRootDirectory` | Drive does not have a root directory | Drive letter exists but root is missing |

```csharp
DriveInfo drive = new DriveInfo("C");
if (drive.DriveType == DriveType.Fixed)
{
    Console.WriteLine("This is an internal drive");
}
```

---

## Formatting Bytes to Human-Readable Sizes

Raw byte values from `TotalSize` and `AvailableFreeSpace` are hard to read. Here is a reusable helper:

```csharp
static string FormatBytes(long bytes)
{
    string[] suffixes = { "B", "KB", "MB", "GB", "TB", "PB" };
    int order = 0;
    double size = bytes;

    while (size >= 1024 && order < suffixes.Length - 1)
    {
        order++;
        size /= 1024;
    }

    return $"{size:F2} {suffixes[order]}";
}

// Usage:
DriveInfo drive = new DriveInfo("C");
Console.WriteLine(FormatBytes(drive.TotalSize));          // "476.94 GB"
Console.WriteLine(FormatBytes(drive.AvailableFreeSpace)); // "123.45 GB"
```

---

## Practical Example: List All Drives With Details

```csharp
using System;
using System.IO;

Console.WriteLine($"{"Drive",-6} {"Type",-12} {"Format",-8} {"Label",-16} {"Total",>12} {"Free",>12} {"Used %",>8}");
Console.WriteLine(new string('-', 80));

foreach (DriveInfo drive in DriveInfo.GetDrives())
{
    // Always check IsReady -- removable drives may not have media inserted
    if (!drive.IsReady)
    {
        Console.WriteLine($"{drive.Name,-6} {"(not ready)",-12}");
        continue;
    }

    double usedPercent = drive.TotalSize > 0
        ? (1.0 - (double)drive.AvailableFreeSpace / drive.TotalSize) * 100
        : 0;

    Console.WriteLine(
        $"{drive.Name,-6} " +
        $"{drive.DriveType,-12} " +
        $"{drive.DriveFormat,-8} " +
        $"{drive.VolumeLabel,-16} " +
        $"{FormatBytes(drive.TotalSize),>12} " +
        $"{FormatBytes(drive.AvailableFreeSpace),>12} " +
        $"{usedPercent,>7:F1}%"
    );
}

static string FormatBytes(long bytes)
{
    string[] suffixes = { "B", "KB", "MB", "GB", "TB", "PB" };
    int order = 0;
    double size = bytes;
    while (size >= 1024 && order < suffixes.Length - 1)
    {
        order++;
        size /= 1024;
    }
    return $"{size:F2} {suffixes[order]}";
}
```

Sample output:

```
Drive  Type         Format   Label                   Total         Free    Used %
--------------------------------------------------------------------------------
C:\    Fixed        NTFS     Windows               476.94 GB    123.45 GB   74.1%
D:\    Fixed        NTFS     Data                  931.51 GB    456.78 GB   51.0%
E:\    CDRom        (not ready)
F:\    Removable    exFAT    USB_DRIVE              59.63 GB     32.10 GB   46.2%
```

---

## Disk Space Check Before Writing

A practical use case -- verify sufficient disk space before writing a large file:

```csharp
void EnsureDiskSpace(string targetPath, long requiredBytes)
{
    string? root = Path.GetPathRoot(Path.GetFullPath(targetPath));
    if (root is null)
        throw new ArgumentException("Cannot determine drive root", nameof(targetPath));

    DriveInfo drive = new DriveInfo(root);

    if (!drive.IsReady)
        throw new IOException($"Drive {drive.Name} is not ready");

    if (drive.AvailableFreeSpace < requiredBytes)
    {
        throw new IOException(
            $"Insufficient disk space on {drive.Name}. " +
            $"Required: {FormatBytes(requiredBytes)}, " +
            $"Available: {FormatBytes(drive.AvailableFreeSpace)}");
    }
}

// Usage:
long fileSize = 500L * 1024 * 1024; // 500 MB
EnsureDiskSpace(@"D:\Backups\dump.sql", fileSize);
```

---

## Summary

- `DriveInfo` queries drive capacity, free space, format, type, and volume label.
- `DriveInfo.GetDrives()` returns all drives on the system. Construct `new DriveInfo("C")` for a specific drive.
- **Always check `IsReady`** before accessing size/format properties -- unready drives throw `IOException`.
- Use `AvailableFreeSpace` (not `TotalFreeSpace`) when checking whether the current user has enough space to write.
- `DriveType` enum distinguishes Fixed, Removable, Network, CDRom, and other drive types.
- Format raw byte values into human-readable strings (GB, MB) for display.
