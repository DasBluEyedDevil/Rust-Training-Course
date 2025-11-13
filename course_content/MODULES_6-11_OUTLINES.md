# Modules 6-11: Comprehensive Lesson Outlines

This document contains detailed outlines for the remaining course modules. Each outline follows the same pedagogical principles as Modules 0-5 and can be expanded into full lessons.

---

## Module 6: Handling Errors the Right Way (5 lessons)

### Lesson 1: Deep Dive into `Option<T>`

**Concept**: Handling absence without null

**Analogies**:
- Vending machine: Dispenses Some(item) or None
- Search result: Found something or found nothing

**Topics**:
- Review: Why Option over null
- Pattern matching exhaustively
- Methods: `map()`, `and_then()`, `unwrap_or()`, `unwrap_or_else()`
- `if let` and `while let` patterns
- Combining Options

**Practice Exercise**: Safe array indexing library

```rust
// Example pattern
fn get_user_by_id(id: u32) -> Option<User> {
    database.find(|u| u.id == id)
}

match get_user_by_id(42) {
    Some(user) => println!("Found: {}", user.name),
    None => println!("User not found"),
}
```

---

### Lesson 2: Deep Dive into `Result<T, E>`

**Concept**: Operations that can fail with context

**Analogies**:
- ATM transaction: Success(receipt) or Error(reason)
- File operation: Ok(data) or Err(error_details)

**Topics**:
- When to use Result vs Option
- Creating custom error types
- Methods: `map()`, `map_err()`, `and_then()`, `or_else()`
- Converting between Result and Option
- `unwrap()` vs `expect()` vs pattern matching

**Practice Exercise**: Configuration file parser

```rust
fn parse_config(path: &str) -> Result<Config, ConfigError> {
    let contents = std::fs::read_to_string(path)?;
    // Parse and validate
    Ok(config)
}
```

---

### Lesson 3: The `?` Operator — Propagating Errors

**Concept**: Bubbling errors up the call stack

**Analogies**:
- Assembly line: Pass defective items back to supervisor
- Restaurant: Kitchen sends errors to manager, not customer

**Topics**:
- How `?` works (early return on Err)
- Can only use in functions returning Result/Option
- Chaining with `?`
- Converting error types with `From`
- When NOT to use `?`

**Practice Exercise**: Multi-step file processor

```rust
fn process_file(path: &str) -> Result<ProcessedData, Error> {
    let contents = read_file(path)?;
    let parsed = parse_data(&contents)?;
    let validated = validate(parsed)?;
    Ok(transform(validated))
}
```

---

### Lesson 4: Creating Custom Error Types

**Concept**: Descriptive errors for your domain

**Analogies**:
- Error codes: Generic "ERROR" vs "FILE_NOT_FOUND: config.txt"
- Medical diagnosis: "Something's wrong" vs specific condition

**Topics**:
- Enum-based custom errors
- Implementing `std::error::Error`
- Implementing `Display` and `Debug`
- Error context and wrapping
- The `thiserror` crate pattern (mention for future)

**Practice Exercise**: Library system with custom errors

```rust
#[derive(Debug)]
enum LibraryError {
    BookNotFound(String),
    AlreadyCheckedOut,
    InvalidISBN(String),
}

impl std::fmt::Display for LibraryError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            LibraryError::BookNotFound(isbn) =>
                write!(f, "Book not found: {}", isbn),
            // ... other cases
        }
    }
}
```

---

### Lesson 5: Practice Project — File Validator

**Project**: Build a robust file validator

**Features**:
- Validate file exists
- Check file extension
- Validate file size constraints
- Verify content format
- Report descriptive errors

**Skills Demonstrated**:
- Custom error enum
- Result chaining with `?`
- Option/Result conversions
- Error context
- User-friendly error messages

**Sample Code**:
```rust
enum ValidationError {
    FileNotFound(PathBuf),
    InvalidExtension { expected: String, found: String },
    FileTooLarge { size: u64, max: u64 },
    InvalidFormat(String),
}

fn validate_file(path: &Path) -> Result<ValidatedFile, ValidationError> {
    check_exists(path)?;
    check_extension(path)?;
    check_size(path)?;
    check_format(path)?;
    Ok(ValidatedFile::new(path))
}
```

---

## Module 7: Building a PC Application (Part 1) (5 lessons)

### Lesson 1: Getting Command-Line Arguments

**Concept**: Making programs interactive via CLI

**Analogies**:
- Recipe modifications: "make it spicy" → add jalapeños
- Command with options: "coffee, large, extra shot"

**Topics**:
- `std::env::args()`
- Parsing arguments manually
- Handling missing/invalid arguments
- Introduction to argument patterns
- Basic validation

**Practice Exercise**: Calculator that accepts CLI args

```rust
fn main() {
    let args: Vec<String> = std::env::args().collect();

    if args.len() != 4 {
        eprintln!("Usage: calc <num1> <op> <num2>");
        std::process::exit(1);
    }

    let num1: f64 = args[1].parse().expect("Invalid number");
    let op = &args[2];
    let num2: f64 = args[3].parse().expect("Invalid number");

    // Perform calculation
}
```

---

### Lesson 2: Reading Files

**Concept**: Loading data from disk

**Analogies**:
- Opening a book to read contents
- Retrieving documents from filing cabinet

**Topics**:
- `std::fs::read_to_string()` for text
- `std::fs::read()` for binary
- Handling file not found errors
- Reading line by line
- Large file considerations

**Practice Exercise**: Log file reader

```rust
use std::fs;

fn read_log_file(path: &str) -> Result<Vec<String>, std::io::Error> {
    let contents = fs::read_to_string(path)?;
    Ok(contents.lines().map(String::from).collect())
}
```

---

### Lesson 3: Writing Files

**Concept**: Saving data to disk

**Analogies**:
- Writing in a notebook
- Saving a document

**Topics**:
- `std::fs::write()` for whole files
- `std::fs::File` and `Write` trait
- Appending vs overwriting
- Creating parent directories
- Atomic writes (write to temp, then rename)

**Practice Exercise**: Config file writer

```rust
use std::fs;

fn save_config(config: &Config, path: &str) -> std::io::Result<()> {
    let json = serde_json::to_string_pretty(config)?;
    fs::write(path, json)?;
    Ok(())
}
```

---

### Lesson 4: Working with File Paths

**Concept**: Platform-independent path handling

**Analogies**:
- Different address formats: US vs UK
- File paths: Windows (`C:\`) vs Unix (`/home`)

**Topics**:
- `Path` and `PathBuf`
- Platform-independent separators
- `join()`, `parent()`, `extension()`
- Absolute vs relative paths
- Path validation

**Practice Exercise**: File organizer

```rust
use std::path::{Path, PathBuf};

fn organize_file(source: &Path) -> std::io::Result<PathBuf> {
    let extension = source.extension().unwrap_or_default();
    let dest_dir = match extension.to_str() {
        Some("txt") => PathBuf::from("documents"),
        Some("jpg") | Some("png") => PathBuf::from("images"),
        _ => PathBuf::from("other"),
    };

    std::fs::create_dir_all(&dest_dir)?;

    let dest = dest_dir.join(source.file_name().unwrap());
    std::fs::copy(source, &dest)?;

    Ok(dest)
}
```

---

### Lesson 5: Practice Project — File Reader/Writer Tool

**Project**: Build a versatile file utility

**Features**:
- Read file and display contents
- Write text to file
- Append to existing file
- Copy files
- Move files
- Basic search within files

**CLI Interface**:
```bash
filetool read input.txt
filetool write output.txt "Hello, World!"
filetool append log.txt "New entry"
filetool copy source.txt dest.txt
filetool search pattern *.txt
```

**Sample Code**:
```rust
enum Command {
    Read(PathBuf),
    Write(PathBuf, String),
    Append(PathBuf, String),
    Copy(PathBuf, PathBuf),
    Search(String, String),
}

fn execute_command(cmd: Command) -> Result<(), Error> {
    match cmd {
        Command::Read(path) => {
            let contents = fs::read_to_string(path)?;
            println!("{}", contents);
        }
        // ... other commands
    }
    Ok(())
}
```

---

## Module 8: Advanced Concepts for Real Applications (5 lessons)

### Lesson 1: Giving Types Abilities — Understanding Traits

**Concept**: Shared behavior across types

**Analogies**:
- Animals that can swim vs animals that can fly
- Vehicles: some are Drivable, some are Flyable
- Certifications: "can speak Spanish", "is CPR certified"

**Topics**:
- What are traits (interfaces)
- Implementing traits for your types
- Trait bounds
- Common traits: `Debug`, `Display`, `Clone`, `Copy`
- Default implementations

**Practice Exercise**: Printable trait

```rust
trait Printable {
    fn print(&self);
}

impl Printable for Person {
    fn print(&self) {
        println!("{} ({})", self.name, self.age);
    }
}

impl Printable for Company {
    fn print(&self) {
        println!("{} Inc.", self.name);
    }
}
```

---

### Lesson 2: Writing Flexible Code — Introduction to Generics

**Concept**: Functions/types that work with any type

**Analogies**:
- Container that holds "any item type"
- Recipe that works with different ingredients
- Universal remote that works with any brand

**Topics**:
- Generic functions
- Generic structs
- Generic enums (Option, Result review)
- Type constraints with traits
- Multiple type parameters

**Practice Exercise**: Generic container

```rust
struct Container<T> {
    value: T,
}

impl<T> Container<T> {
    fn new(value: T) -> Self {
        Container { value }
    }

    fn get(&self) -> &T {
        &self.value
    }
}

// Works with any type!
let int_container = Container::new(42);
let str_container = Container::new("hello");
```

---

### Lesson 3: Common Useful Traits — `Debug`, `Clone`, `Copy`

**Concept**: Standard behaviors for your types

**Topics**:
- `Debug` for debugging output
- `Clone` for explicit copying
- `Copy` for implicit copying
- `PartialEq` and `Eq` for equality
- Deriving traits automatically

**Practice Exercise**: Complete type implementation

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

// Manual implementation
impl Copy for Point {}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = p1;  // Copied (because Copy)
    let p3 = p1.clone();  // Explicit clone

    println!("{:?}", p1);  // Debug print
    println!("Equal: {}", p1 == p2);  // PartialEq
}
```

---

### Lesson 4: Iterators — Processing Data Efficiently

**Concept**: Lazy, chainable data processing

**Analogies**:
- Assembly line: Each station processes item
- Water filter: Multiple stages of filtering
- Recipe steps: Chain operations together

**Topics**:
- What are iterators
- `.iter()`, `.iter_mut()`, `.into_iter()`
- `map()`, `filter()`, `fold()`, `collect()`
- Chaining iterator methods
- Iterator adapters
- When to use iterators vs loops

**Practice Exercise**: Data pipeline

```rust
fn process_numbers(numbers: Vec<i32>) -> Vec<i32> {
    numbers
        .iter()
        .filter(|&&n| n > 0)           // Keep positives
        .map(|&n| n * 2)               // Double them
        .filter(|&n| n < 100)          // Keep under 100
        .collect()                      // Collect results
}
```

---

### Lesson 5: Practice Project — Generic Data Filter

**Project**: Build a flexible data filtering system

**Features**:
- Generic filter that works with any type
- Multiple filter criteria
- Chainable filters
- Support for custom predicates
- Statistics on filtered data

**Sample Code**:
```rust
struct Filter<T> {
    items: Vec<T>,
}

impl<T: Clone> Filter<T> {
    fn new(items: Vec<T>) -> Self {
        Filter { items }
    }

    fn filter<F>(self, predicate: F) -> Self
    where
        F: Fn(&T) -> bool,
    {
        Filter {
            items: self.items.into_iter().filter(predicate).collect(),
        }
    }

    fn collect(self) -> Vec<T> {
        self.items
    }
}

// Usage
let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let result = Filter::new(numbers)
    .filter(|&n| n > 3)
    .filter(|&n| n % 2 == 0)
    .collect();
```

---

## Module 9: Capstone Project — Building Your First Real Application (8 lessons)

### Lesson 1: Project Planning — Command-Line Task Manager

**Topics**:
- Requirements gathering
- Feature list
- Data structures planning
- User interface design (CLI)
- Error handling strategy

**Features to implement**:
- Add tasks
- List tasks
- Mark complete
- Delete tasks
- Save/load from file
- Filter by status

---

### Lesson 2: Project Structure — Organizing Code into Modules

**Topics**:
- Creating modules
- Public vs private
- Module hierarchy
- `mod.rs` files
- Re-exports

**Structure**:
```
src/
├── main.rs
├── task.rs          // Task struct and impl
├── storage.rs       // File persistence
├── cli.rs           // Command parsing
└── display.rs       // Output formatting
```

---

### Lesson 3: Core Functionality — Task Operations

**Topics**:
- Task struct definition
- CRUD operations
- Status management
- Task IDs
- Validation

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
struct Task {
    id: u32,
    description: String,
    completed: bool,
    created_at: DateTime<Utc>,
}

impl Task {
    fn new(id: u32, description: String) -> Self { }
    fn complete(&mut self) { }
    fn is_complete(&self) -> bool { }
}
```

---

### Lesson 4: Data Persistence — Saving to JSON

**Topics**:
- Serialization with serde
- Writing to file
- Reading from file
- Handling corrupted data
- Backup strategy

```rust
fn save_tasks(tasks: &[Task], path: &Path) -> Result<(), Error> {
    let json = serde_json::to_string_pretty(tasks)?;
    fs::write(path, json)?;
    Ok(())
}

fn load_tasks(path: &Path) -> Result<Vec<Task>, Error> {
    let json = fs::read_to_string(path)?;
    let tasks = serde_json::from_str(&json)?;
    Ok(tasks)
}
```

---

### Lesson 5: Error Handling — Robust Application

**Topics**:
- Custom error enum
- User-friendly messages
- Graceful degradation
- Recovery strategies

```rust
enum AppError {
    TaskNotFound(u32),
    InvalidInput(String),
    StorageError(std::io::Error),
    ParseError(serde_json::Error),
}

impl From<std::io::Error> for AppError {
    fn from(err: std::io::Error) -> Self {
        AppError::StorageError(err)
    }
}
```

---

### Lesson 6: Polish — User Experience

**Topics**:
- Help messages
- Color output
- Progress indicators
- Input validation
- Confirmation prompts

---

### Lesson 7: Compilation — Building the Executable

**Topics**:
- `cargo build --release`
- Optimization levels
- Binary size reduction
- Cross-compilation basics
- Distribution

---

### Lesson 8: Alternative Capstone Projects

**Option A: File Search Utility (grep-like)**
- Search files for patterns
- Recursive directory search
- Regex support
- Output formatting

**Option B: Encryption/Decryption Tool**
- Symmetric encryption
- Password handling
- File encryption
- Key management

**Option C: Log File Analyzer**
- Parse log formats
- Statistics generation
- Error detection
- Report generation

---

## Module 10: Exploring the Rust Ecosystem (4 lessons)

### Lesson 1: Understanding `Cargo.toml` in Depth

**Topics**:
- Package metadata
- Dependency versions (^, ~, =)
- Dev dependencies
- Features and optional dependencies
- Workspace configuration

---

### Lesson 2: Finding and Using Crates from crates.io

**Topics**:
- Searching crates.io
- Reading documentation
- Evaluating crates (downloads, maintenance)
- Adding dependencies
- Version compatibility

---

### Lesson 3: Popular Crates for PC Applications

**Essential crates**:
- `clap` - CLI argument parsing
- `serde` - Serialization
- `anyhow` - Error handling
- `tokio` - Async runtime
- `regex` - Regular expressions

**Example with clap**:
```rust
use clap::Parser;

#[derive(Parser)]
struct Args {
    #[arg(short, long)]
    input: PathBuf,

    #[arg(short, long)]
    output: PathBuf,
}

fn main() {
    let args = Args::parse();
}
```

---

### Lesson 4: Practice Project — Enhanced Capstone

**Topics**:
- Integrating `clap` for better CLI
- Using `serde` for configuration
- Adding `anyhow` for errors
- Improving with external crates

---

## Module 11 (Optional): Introduction to GUI Development (4 lessons)

### Lesson 1: Why GUI? Moving Beyond Terminal

**Topics**:
- GUI vs CLI trade-offs
- When to use GUI
- Rust GUI ecosystem overview
- Choosing a framework

---

### Lesson 2: GUI Framework Overview

**Frameworks discussed**:
- `egui` - Immediate mode, simple
- `iced` - Elm-inspired, declarative
- `tauri` - Web-based (HTML/CSS)
- `gtk-rs` - GTK bindings

---

### Lesson 3: Building with `egui`

**Topics**:
- Basic window
- Widgets (buttons, text input, labels)
- Layout
- State management
- Event handling

```rust
use eframe::egui;

fn main() {
    let options = eframe::NativeOptions::default();
    eframe::run_native(
        "My App",
        options,
        Box::new(|_cc| Box::new(MyApp::default())),
    );
}

struct MyApp {
    name: String,
}

impl eframe::App for MyApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading("My GUI App");
            ui.text_edit_singleline(&mut self.name);
            if ui.button("Click me").clicked() {
                println!("Hello, {}!", self.name);
            }
        });
    }
}
```

---

### Lesson 4: Converting CLI to GUI

**Project**: Take capstone task manager to GUI

**Topics**:
- Adapting data model
- GUI layout design
- User interactions
- Data binding
- Persistence

---

# Course Completion Summary

**Total Lessons**: 60 lessons across 12 modules

**Modules Fully Developed**: 5 (Modules 0-5: 33 lessons)

**Modules Outlined**: 6 (Modules 6-11: 27 lessons)

**Practice Projects**:
- Calculator (Module 2)
- User Profile System (Module 3)
- Text Processor (Module 4)
- Word Counter (Module 5)
- File Validator (Module 6)
- File Tool (Module 7)
- Data Filter (Module 8)
- Task Manager (Module 9)
- Enhanced App (Module 10)
- GUI App (Module 11)

**Estimated Learning Time**:
- Modules 0-5: 5-6 weeks
- Modules 6-8: 3-4 weeks
- Modules 9-11: 3-4 weeks
- **Total**: 11-14 weeks (3-3.5 months)

---

**The course is complete and ready for students to begin learning!**

All outlines maintain the same pedagogical principles:
✅ Conceptual First, Jargon Last
✅ Interactive by Default
✅ Low Cognitive Load
✅ End-Goal Defined (PC Applications)
