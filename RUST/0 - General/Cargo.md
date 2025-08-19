---
tags:
  - rust
  - fundamental
---

**Cargo** is the **official build system and package manager** for Rust. Think of it as a combination of tools like:

- npm for JavaScript (manages packages),
- make or cmake for C/C++ (build system),
- and even a bit of git-like project scaffolding and dependency resolution.
  

It’s included by default when you install Rust using rustup.

---

### **🧱 What Does Cargo Actually Do?**

1. **Manages Your Code Project**
    
    - When you start a new Rust project with cargo new, it creates a directory with all the files you need, like:
        
        - Cargo.toml – your project’s configuration and dependency list.
            
        - src/main.rs – your main code file.
            
        
    - This helps enforce a consistent structure for Rust projects.
        
    
2. **Compiles Your Code**
    
    - It handles all the complex steps of turning .rs files into a binary.
        
    - You don’t need to run rustc manually.
        
    - It compiles dependencies **only once** (caching them) for faster rebuilds.
        
    
3. **Manages Dependencies**
    
    - You list your required libraries (called _crates_) in Cargo.toml.
        
    - Cargo fetches them from [crates.io](https://crates.io) automatically.
        
    - Handles version resolution, updates, and local caching.
        
    
4. **Runs and Tests Code**
    
    - You can run your application with cargo run (no need to manually compile and execute).
        
    - You can run your tests with cargo test, which automatically finds and executes all test functions.
        
    
5. **Build Profiles**
    
    - Cargo builds in **debug mode** by default (fast compilation, no optimizations).
        
    - It can build in **release mode** (optimized for speed) with --release.
        
    
6. **Generates Documentation**
    
    - It can build HTML documentation using rustdoc for your code and dependencies.
        
    - Helps developers understand their own and others’ code better.
        
    
7. **Enables Publishing**
    
    - If you build a useful crate, you can share it via [crates.io](https://crates.io) using Cargo.
        
    

---

### **📌 Why is Cargo Important?**

- **Standardized Workflow**: Every Rust project uses Cargo. This consistency reduces onboarding time and confusion.
    
- **Tight Integration with Rust**: Cargo knows how to work with the Rust compiler and the ecosystem.
    
- **Productivity**: It handles everything from testing to documentation in one tool.
    
- **Speed & Efficiency**: It caches builds and dependencies, making development fast.
    

---

### **🧭 Typical Cargo Project Structure**

```
my_project/
├── Cargo.toml        # Package config: name, version, dependencies
├── Cargo.lock        # Exact versions of dependencies (auto-generated)
└── src/
    └── main.rs       # Your application entry point
```

For libraries, the main file is usually lib.rs instead of main.rs.

---

If you’re coming from another language:

- Cargo is to Rust what **npm** is to Node.js or **pip + setuptools** is to Python.