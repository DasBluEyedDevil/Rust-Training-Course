# Module 6, Lesson 5: Practice Project — Building a File Validator

## Project Overview

Build a professional file validation system that demonstrates all error handling techniques!

**What we're building:**
- Validate file existence, size, format
- Parse and validate file contents
- Comprehensive error reporting
- Clean error propagation with `?`

**Skills practiced:**
- ✅ `Option<T>` for optional values
- ✅ `Result<T, E>` for operations that can fail
- ✅ The `?` operator for clean error propagation
- ✅ Custom error types with detailed information
- ✅ Error conversion with `From` trait
- ✅ All error handling patterns working together

## The Complete Project

### **Step 1: Create the Project**

```bash
cargo new file_validator
cd file_validator
code .
```

### **Step 2: Build the Validator (Complete Code)**

```rust
use std::error::Error;
use std::fmt;
use std::fs;
use std::io;
use std::path::Path;

// ===== Custom Error Types =====

#[derive(Debug)]
enum ValidationError {
    FileNotFound(String),
    FileTooLarge { path: String, size: u64, max: u64 },
    FileTooSmall { path: String, size: u64, min: u64 },
    InvalidExtension { path: String, expected: Vec<String> },
    EmptyFile(String),
    InvalidFormat(String),
    ParseError { line: usize, reason: String },
    IoError(io::Error),
}

impl fmt::Display for ValidationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ValidationError::FileNotFound(path) => {
                write!(f, "File not found: {}", path)
            }
            ValidationError::FileTooLarge { path, size, max } => {
                write!(f, "File '{}' is too large: {} bytes (max: {})", path, size, max)
            }
            ValidationError::FileTooSmall { path, size, min } => {
                write!(f, "File '{}' is too small: {} bytes (min: {})", path, size, min)
            }
            ValidationError::InvalidExtension { path, expected } => {
                write!(
                    f,
                    "File '{}' has wrong extension. Expected: {:?}",
                    path, expected
                )
            }
            ValidationError::EmptyFile(path) => {
                write!(f, "File '{}' is empty", path)
            }
            ValidationError::InvalidFormat(msg) => {
                write!(f, "Invalid format: {}", msg)
            }
            ValidationError::ParseError { line, reason } => {
                write!(f, "Parse error at line {}: {}", line, reason)
            }
            ValidationError::IoError(e) => {
                write!(f, "IO error: {}", e)
            }
        }
    }
}

impl Error for ValidationError {}

impl From<io::Error> for ValidationError {
    fn from(error: io::Error) -> Self {
        ValidationError::IoError(error)
    }
}

// ===== Validation Rules =====

struct ValidationRules {
    allowed_extensions: Vec<String>,
    min_size: u64,
    max_size: u64,
    require_header: Option<String>,
}

impl ValidationRules {
    fn for_csv() -> Self {
        ValidationRules {
            allowed_extensions: vec![String::from("csv")],
            min_size: 1,
            max_size: 10_000_000,  // 10 MB
            require_header: Some(String::from("name,age,email")),
        }
    }

    fn for_text() -> Self {
        ValidationRules {
            allowed_extensions: vec![String::from("txt"), String::from("md")],
            min_size: 1,
            max_size: 1_000_000,  // 1 MB
            require_header: None,
        }
    }
}

// ===== File Validator =====

struct FileValidator {
    path: String,
    rules: ValidationRules,
}

impl FileValidator {
    fn new(path: &str, rules: ValidationRules) -> Self {
        FileValidator {
            path: path.to_string(),
            rules,
        }
    }

    fn validate(&self) -> Result<ValidationReport, ValidationError> {
        // Check existence
        self.check_exists()?;

        // Check size
        let size = self.check_size()?;

        // Check extension
        self.check_extension()?;

        // Read and validate content
        let content = self.read_content()?;
        let line_count = self.validate_content(&content)?;

        Ok(ValidationReport {
            path: self.path.clone(),
            size,
            line_count,
            status: String::from("Valid"),
        })
    }

    fn check_exists(&self) -> Result<(), ValidationError> {
        if !Path::new(&self.path).exists() {
            return Err(ValidationError::FileNotFound(self.path.clone()));
        }
        Ok(())
    }

    fn check_size(&self) -> Result<u64, ValidationError> {
        let metadata = fs::metadata(&self.path)?;
        let size = metadata.len();

        if size == 0 {
            return Err(ValidationError::EmptyFile(self.path.clone()));
        }

        if size < self.rules.min_size {
            return Err(ValidationError::FileTooSmall {
                path: self.path.clone(),
                size,
                min: self.rules.min_size,
            });
        }

        if size > self.rules.max_size {
            return Err(ValidationError::FileTooLarge {
                path: self.path.clone(),
                size,
                max: self.rules.max_size,
            });
        }

        Ok(size)
    }

    fn check_extension(&self) -> Result<(), ValidationError> {
        let path = Path::new(&self.path);

        let extension = path
            .extension()
            .and_then(|ext| ext.to_str())
            .map(|s| s.to_lowercase());

        match extension {
            Some(ext) => {
                if !self.rules.allowed_extensions.contains(&ext) {
                    return Err(ValidationError::InvalidExtension {
                        path: self.path.clone(),
                        expected: self.rules.allowed_extensions.clone(),
                    });
                }
                Ok(())
            }
            None => Err(ValidationError::InvalidExtension {
                path: self.path.clone(),
                expected: self.rules.allowed_extensions.clone(),
            }),
        }
    }

    fn read_content(&self) -> Result<String, ValidationError> {
        let content = fs::read_to_string(&self.path)?;
        Ok(content)
    }

    fn validate_content(&self, content: &str) -> Result<usize, ValidationError> {
        let lines: Vec<&str> = content.lines().collect();

        if lines.is_empty() {
            return Err(ValidationError::EmptyFile(self.path.clone()));
        }

        // Check header if required
        if let Some(expected_header) = &self.rules.require_header {
            let actual_header = lines[0].trim();
            if actual_header != expected_header {
                return Err(ValidationError::InvalidFormat(format!(
                    "Expected header: {}, got: {}",
                    expected_header, actual_header
                )));
            }
        }

        Ok(lines.len())
    }
}

// ===== Validation Report =====

#[derive(Debug)]
struct ValidationReport {
    path: String,
    size: u64,
    line_count: usize,
    status: String,
}

impl ValidationReport {
    fn print(&self) {
        println!("╔═══════════════════════════════════════╗");
        println!("║       File Validation Report          ║");
        println!("╠═══════════════════════════════════════╣");
        println!("║ File: {:30} ║", self.path);
        println!("║ Size: {:30} bytes ║", self.size);
        println!("║ Lines: {:29} ║", self.line_count);
        println!("║ Status: {:28} ║", self.status);
        println!("╚═══════════════════════════════════════╝");
    }
}

// ===== CSV Parser =====

#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
    email: String,
}

impl Person {
    fn from_csv_line(line: &str, line_num: usize) -> Result<Self, ValidationError> {
        let parts: Vec<&str> = line.split(',').collect();

        if parts.len() != 3 {
            return Err(ValidationError::ParseError {
                line: line_num,
                reason: format!("Expected 3 fields, got {}", parts.len()),
            });
        }

        let name = parts[0].trim().to_string();
        if name.is_empty() {
            return Err(ValidationError::ParseError {
                line: line_num,
                reason: String::from("Name cannot be empty"),
            });
        }

        let age = parts[1]
            .trim()
            .parse::<u32>()
            .map_err(|_| ValidationError::ParseError {
                line: line_num,
                reason: format!("Invalid age: {}", parts[1]),
            })?;

        let email = parts[2].trim().to_string();
        if !email.contains('@') {
            return Err(ValidationError::ParseError {
                line: line_num,
                reason: format!("Invalid email: {}", email),
            });
        }

        Ok(Person { name, age, email })
    }
}

fn parse_csv_file(path: &str) -> Result<Vec<Person>, ValidationError> {
    let content = fs::read_to_string(path)?;
    let mut people = Vec::new();

    for (i, line) in content.lines().enumerate() {
        if i == 0 {
            continue;  // Skip header
        }

        if line.trim().is_empty() {
            continue;  // Skip empty lines
        }

        let person = Person::from_csv_line(line, i + 1)?;
        people.push(person);
    }

    Ok(people)
}

// ===== Main Program =====

fn main() -> Result<(), Box<dyn Error>> {
    println!("=== File Validator Demo ===\n");

    // Create test files
    create_test_files()?;

    // Test 1: Valid CSV file
    println!("Test 1: Validating valid CSV file");
    validate_and_report("data.csv", ValidationRules::for_csv());

    // Test 2: Invalid CSV (wrong header)
    println!("\nTest 2: Validating CSV with wrong header");
    validate_and_report("bad_header.csv", ValidationRules::for_csv());

    // Test 3: File too large
    println!("\nTest 3: Validating file that's too large");
    let mut rules = ValidationRules::for_text();
    rules.max_size = 10;  // Set very low limit
    validate_and_report("large.txt", rules);

    // Test 4: Non-existent file
    println!("\nTest 4: Validating non-existent file");
    validate_and_report("missing.csv", ValidationRules::for_csv());

    // Test 5: Parse CSV and display
    println!("\nTest 5: Parsing and displaying CSV data");
    match parse_csv_file("data.csv") {
        Ok(people) => {
            println!("✅ Successfully parsed {} people:", people.len());
            for person in people {
                println!("  - {} ({} years old) - {}", person.name, person.age, person.email);
            }
        }
        Err(e) => println!("❌ Error: {}", e),
    }

    Ok(())
}

fn validate_and_report(path: &str, rules: ValidationRules) {
    let validator = FileValidator::new(path, rules);

    match validator.validate() {
        Ok(report) => {
            println!("✅ Validation successful!");
            report.print();
        }
        Err(e) => {
            println!("❌ Validation failed: {}", e);
        }
    }
}

fn create_test_files() -> Result<(), Box<dyn Error>> {
    // Valid CSV
    fs::write(
        "data.csv",
        "name,age,email\nAlice,30,alice@example.com\nBob,25,bob@example.com\nCharlie,35,charlie@example.com",
    )?;

    // CSV with wrong header
    fs::write(
        "bad_header.csv",
        "username,years,contact\nAlice,30,alice@example.com",
    )?;

    // Large text file
    fs::write("large.txt", "This is a test file with some content.")?;

    Ok(())
}
```

### **Step 3: Run the Project**

```bash
cargo run
```

**Expected output:**

```
=== File Validator Demo ===

Test 1: Validating valid CSV file
✅ Validation successful!
╔═══════════════════════════════════════╗
║       File Validation Report          ║
╠═══════════════════════════════════════╣
║ File: data.csv                         ║
║ Size: 104                        bytes ║
║ Lines: 4                               ║
║ Status: Valid                          ║
╚═══════════════════════════════════════╝

Test 2: Validating CSV with wrong header
❌ Validation failed: Invalid format: Expected header: name,age,email, got: username,years,contact

Test 3: Validating file that's too large
❌ Validation failed: File 'large.txt' is too large: 39 bytes (max: 10)

Test 4: Validating non-existent file
❌ Validation failed: File not found: missing.csv

Test 5: Parsing and displaying CSV data
✅ Successfully parsed 3 people:
  - Alice (30 years old) - alice@example.com
  - Bob (25 years old) - bob@example.com
  - Charlie (35 years old) - charlie@example.com
```

## Understanding the Code

### **Custom Error Enum**

```rust
#[derive(Debug)]
enum ValidationError {
    FileNotFound(String),
    FileTooLarge { path: String, size: u64, max: u64 },
    // ... more variants with contextual data
}
```

**Why**: Provides specific, actionable error information.

### **Error Conversion**

```rust
impl From<io::Error> for ValidationError {
    fn from(error: io::Error) -> Self {
        ValidationError::IoError(error)
    }
}
```

**Why**: Enables `?` operator to work with different error types.

### **Clean Error Propagation**

```rust
fn validate(&self) -> Result<ValidationReport, ValidationError> {
    self.check_exists()?;         // Return error if fails
    let size = self.check_size()?; // Unwrap if succeeds
    self.check_extension()?;
    // ... continue
}
```

**Why**: Clean, readable code without nested match statements.

### **Option to Result Conversion**

```rust
let extension = path
    .extension()
    .and_then(|ext| ext.to_str())
    .map(|s| s.to_lowercase());

match extension {
    Some(ext) => { /* validate */ },
    None => Err(ValidationError::InvalidExtension { /* ... */ }),
}
```

**Why**: Safely handle potentially missing values.

## Challenges and Extensions

### **Challenge 1: Add JSON Validation**

```rust
// Add to Cargo.toml:
// [dependencies]
// serde_json = "1.0"

use serde_json::Value;

impl ValidationRules {
    fn for_json() -> Self {
        ValidationRules {
            allowed_extensions: vec![String::from("json")],
            min_size: 2,  // At least "{}"
            max_size: 5_000_000,
            require_header: None,
        }
    }
}

fn validate_json(path: &str) -> Result<(), ValidationError> {
    let content = fs::read_to_string(path)?;

    serde_json::from_str::<Value>(&content)
        .map_err(|e| ValidationError::InvalidFormat(format!("Invalid JSON: {}", e)))?;

    Ok(())
}
```

### **Challenge 2: Batch Validation**

```rust
fn validate_multiple(paths: Vec<&str>, rules: ValidationRules) -> Vec<Result<ValidationReport, ValidationError>> {
    paths
        .into_iter()
        .map(|path| FileValidator::new(path, &rules).validate())
        .collect()
}

fn main() {
    let files = vec!["file1.csv", "file2.csv", "file3.csv"];
    let results = validate_multiple(files, ValidationRules::for_csv());

    for (i, result) in results.iter().enumerate() {
        match result {
            Ok(report) => println!("✅ File {}: Valid", i + 1),
            Err(e) => println!("❌ File {}: {}", i + 1, e),
        }
    }
}
```

### **Challenge 3: Warning System**

```rust
#[derive(Debug)]
enum Warning {
    LargeFile(u64),
    ManyLines(usize),
    OldFormat,
}

struct ValidationReport {
    path: String,
    size: u64,
    line_count: usize,
    status: String,
    warnings: Vec<Warning>,  // Add warnings
}

impl FileValidator {
    fn generate_warnings(&self, size: u64, line_count: usize) -> Vec<Warning> {
        let mut warnings = Vec::new();

        if size > 5_000_000 {
            warnings.push(Warning::LargeFile(size));
        }

        if line_count > 10_000 {
            warnings.push(Warning::ManyLines(line_count));
        }

        warnings
    }
}
```

### **Challenge 4: Progress Reporting**

```rust
use std::sync::mpsc;
use std::thread;

fn validate_with_progress(paths: Vec<String>) {
    let (tx, rx) = mpsc::channel();
    let total = paths.len();

    thread::spawn(move || {
        for (i, path) in paths.iter().enumerate() {
            let validator = FileValidator::new(path, ValidationRules::for_csv());
            let result = validator.validate();

            tx.send((i + 1, total, result)).unwrap();
        }
    });

    for (current, total, result) in rx {
        println!("Progress: {}/{}", current, total);
        match result {
            Ok(_) => println!("  ✅ Valid"),
            Err(e) => println!("  ❌ Error: {}", e),
        }
    }
}
```

## Key Takeaways

- ✅ Custom error enums provide detailed, specific error information
- ✅ `From` trait enables seamless error conversion with `?`
- ✅ The `?` operator creates clean, readable error handling code
- ✅ Pattern matching on custom errors allows specific handling
- ✅ `Display` trait implementation provides user-friendly messages
- ✅ Combining `Option` and `Result` handles optional and fallible operations
- ✅ Error context (line numbers, file names) makes debugging easier
- ✅ Validation rules encapsulate reusable logic
- ✅ Comprehensive error handling is the foundation of robust applications
- ✅ All error handling techniques work together seamlessly

---

## ✅ Module 6 Complete!

You've mastered Rust error handling:
- ✅ `Option<T>` for values that might not exist
- ✅ `Result<T, E>` for operations that can fail
- ✅ The `?` operator for clean error propagation
- ✅ Custom error types for specific, actionable errors
- ✅ Built a professional file validation system!

**Next: Module 7 — File I/O and Command-Line Arguments**

Total Progress: 38 lessons complete (~63%)

---

[← Back to Module 6](README.md) | [Continue to Module 7 →](../module_07_file_io/)
