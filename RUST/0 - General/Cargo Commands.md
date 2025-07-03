---
tags: rust, command
---

In Rust, [[Cargo|cargo]] is the official package manager and build system. Here’s a **summary of the most commonly used cargo commands** and what they do:

---

### **📦 Project Management**

- cargo new my_project – Create a new Rust project (with Cargo.toml)
    
- cargo init – Initialize a new Cargo project in an existing directory
    

---

### **🛠️ Build & Run**

- cargo build – Compile the current project (debug mode)
    
- cargo build --release – Compile with optimizations (for production)
    
- cargo run – Build and run the project
    
- cargo check – Quickly check for errors without building
    

---

### **🧪 Testing**

- cargo test – Run unit and integration tests
    
- cargo test -- --nocapture – Show output during tests
    

---

### **📚 Documentation**

- cargo doc – Generate documentation for your crate
    
- cargo doc --open – Build and open documentation in a browser
    

---

### **📦 Dependencies**

- cargo update – Update dependencies listed in Cargo.lock
    
- cargo add crate_name – (Requires cargo-edit) Add a dependency
    
- cargo remove crate_name – Remove a dependency
    

---

### **🔍 Information**

- cargo tree – Shows dependency tree (requires cargo-tree)
    
- cargo metadata – Outputs project metadata in JSON
    

---

### **📤 Publishing**

- cargo publish – Publish your crate to [crates.io](https://crates.io)
    
- cargo login – Login with your crates.io API token
    

---

### **🧰 Other Tools**

- cargo fmt – Format Rust code (requires rustfmt)
    
- cargo clippy – Lint the code for common mistakes (requires clippy)
    