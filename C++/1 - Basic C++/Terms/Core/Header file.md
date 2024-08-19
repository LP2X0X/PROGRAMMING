- C++ code files (with a .cpp extension) are not the only files commonly seen in C++ programs. The other type of file is called a **header file**. Header files usually have a .h extension, but you will occasionally see them with a .hpp extension or no extension at all.
- Example flow:
![[IncludeHeader.webp|center]]

- Source files should include their paired header, this allows the compiler to catch certain kinds of errors at compile time instead of link time.