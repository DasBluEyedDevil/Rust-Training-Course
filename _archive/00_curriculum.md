# Complete Rust Training Course Curriculum

**Target Audience**: Absolute beginners with zero programming experience
**End Goal**: Build and deploy secure, high-performance standalone PC applications

---

## Module 0: Setting Up Your Rust Workshop

- Lesson 0.1: Installing Rust (rustup and cargo)
- Lesson 0.2: Setting up Visual Studio Code with rust-analyzer
- Lesson 0.3: Your First "Hello, World!" - Making the computer say hello
- Lesson 0.4: Understanding Cargo - Your project manager

---

## Module 1: The Rust Language Basics

- Lesson 1.1: The `main()` function - Where your program starts
- Lesson 1.2: Running your code with `cargo run`
- Lesson 1.3: Variables and Boxes - Storing information (`let`, `mut`)
- Lesson 1.4: Numbers and Text - The basic types of data
- Lesson 1.5: Functions - Teaching your program new tricks
- Lesson 1.6: Comments - Leaving notes for yourself

---

## Module 2: Making Decisions and Repeating Actions

- Lesson 2.1: Making choices - The `if/else` decision maker
- Lesson 2.2: Repeating forever (until you say stop) - The `loop`
- Lesson 2.3: Repeating while something is true - The `while` loop
- Lesson 2.4: Counting through a list - The `for` loop
- Lesson 2.5: Practice Project: Building a simple calculator

---

## Module 3: Organizing Your Data

- Lesson 3.1: Grouping related information - Introduction to Structs
- Lesson 3.2: Multiple possibilities - Introduction to Enums
- Lesson 3.3: When something might not exist - The `Option` enum
- Lesson 3.4: When things can go wrong - The `Result` enum
- Lesson 3.5: Practice Project: Creating a simple user profile system

---

## Module 4: The Ownership System (Rust's Superpower)

- Lesson 4.1: Where does data live? - The Stack and The Heap (with analogies)
- Lesson 4.2: Who owns this data? - Understanding Ownership rules
- Lesson 4.3: Fighting the compiler (on purpose) - Common ownership errors
- Lesson 4.4: Lending data temporarily - Immutable borrowing (references)
- Lesson 4.5: Lending with permission to change - Mutable borrowing
- Lesson 4.6: Fighting the borrow checker - Common borrowing errors
- Lesson 4.7: How long can I use this? - Understanding Lifetimes
- Lesson 4.8: Practice Project: Building a text processor that doesn't copy data

---

## Module 5: Working with Collections of Data

- Lesson 5.1: Text you own vs. text you borrow - `String` vs `&str`
- Lesson 5.2: Growing lists - Vectors (`Vec<T>`)
- Lesson 5.3: Looking things up by name - Hash Maps (`HashMap<K, V>`)
- Lesson 5.4: Practice Project: Building a word counter

---

## Module 6: Handling Errors the Right Way

- Lesson 6.1: When things might not exist - Deep dive into `Option<T>`
- Lesson 6.2: When operations might fail - Deep dive into `Result<T, E>`
- Lesson 6.3: Propagating errors easily - The `?` operator
- Lesson 6.4: Creating your own error messages
- Lesson 6.5: Practice Project: Building a robust file validator

---

## Module 7: Building a PC Application (Part 1)

- Lesson 7.1: Getting user input from the command line - `std::env::args()`
- Lesson 7.2: Reading the contents of a file - `std::fs::read_to_string()`
- Lesson 7.3: Writing data to a file - `std::fs::write()`
- Lesson 7.4: Working with file paths - The `Path` and `PathBuf` types
- Lesson 7.5: Practice Project: Building a simple file reader/writer tool

---

## Module 8: Advanced Concepts for Real Applications

- Lesson 8.1: Giving types abilities - Understanding Traits
- Lesson 8.2: Writing flexible code - Introduction to Generics
- Lesson 8.3: Common useful traits - `Debug`, `Clone`, `Copy`
- Lesson 8.4: Iterators - Processing data efficiently
- Lesson 8.5: Practice Project: Building a generic data filter

---

## Module 9: Capstone Project - Building Your First Real Application

- Lesson 9.1: Project Planning - Designing a command-line task manager
- Lesson 9.2: Project Structure - Organizing code into modules
- Lesson 9.3: Core Functionality - Adding, listing, and completing tasks
- Lesson 9.4: Data Persistence - Saving tasks to a file (JSON format)
- Lesson 9.5: Error Handling - Making the app robust
- Lesson 9.6: Polish - Adding help messages and user-friendly output
- Lesson 9.7: Compilation - Building your standalone executable
- Lesson 9.8: Alternative Capstone Options:
  - Option A: File search utility (grep-like tool)
  - Option B: Simple encryption/decryption tool
  - Option C: Log file analyzer

---

## Module 10: Exploring the Rust Ecosystem

- Lesson 10.1: Understanding `Cargo.toml` - Your project's configuration
- Lesson 10.2: Finding and using external crates from crates.io
- Lesson 10.3: Popular crates for PC applications:
  - `clap` - Command-line argument parsing
  - `serde` - Serialization/deserialization
  - `anyhow` - Better error handling
- Lesson 10.4: Practice Project: Enhancing your capstone with external crates

---

## Module 11 (Optional): Introduction to GUI Development

- Lesson 11.1: Why GUI? - Moving beyond the terminal
- Lesson 11.2: Choosing a GUI framework - Brief overview of `egui`, `iced`, `tauri`
- Lesson 11.3: Building your first window with `egui`
- Lesson 11.4: Converting your CLI app to GUI (basic version)

---

## Course Statistics

- **Total Modules**: 12 (including optional Module 11)
- **Total Estimated Lessons**: ~60 lessons
- **Practice Projects**: 10+ hands-on projects
- **Capstone Projects**: 1 major + 3 alternatives
- **Focus**: 100% oriented toward standalone PC application development
