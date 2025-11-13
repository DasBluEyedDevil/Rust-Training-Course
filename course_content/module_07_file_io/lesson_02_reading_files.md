# Module 7, Lesson 2: Loading Data from Disk — Reading Files

## The Concept: Opening a Book

Imagine reading a book:
- **Open it**: Access the contents
- **Read pages**: Get the text
- **Close it**: Done automatically in Rust!

**File reading** lets your program load and process data stored on disk.

## The Simplest Way: read_to_string()

```rust
use std::fs;

fn main() {
    let content = fs::read_to_string("hello.txt")
        .expect("Failed to read file");

    println!("File contents:\n{}", content);
}
```

**Create hello.txt first:**
```bash
echo "Hello from a file!" > hello.txt
```

**Run:**
```bash
cargo run
```

**Output:**
```
File contents:
Hello from a file!
```

## Proper Error Handling

```rust
use std::fs;

fn main() {
    match fs::read_to_string("hello.txt") {
        Ok(content) => {
            println!("✅ File read successfully!");
            println!("Contents:\n{}", content);
        }
        Err(error) => {
            println!("❌ Error reading file: {}", error);
        }
    }
}
```

## With the ? Operator

```rust
use std::error::Error;
use std::fs;

fn read_file(path: &str) -> Result<String, Box<dyn Error>> {
    let content = fs::read_to_string(path)?;
    Ok(content)
}

fn main() -> Result<(), Box<dyn Error>> {
    let content = read_file("hello.txt")?;
    println!("File contents:\n{}", content);
    Ok(())
}
```

## Reading Line by Line

For large files, read line by line:

```rust
use std::fs::File;
use std::io::{BufRead, BufReader};

fn main() -> std::io::Result<()> {
    let file = File::open("hello.txt")?;
    let reader = BufReader::new(file);

    for (i, line) in reader.lines().enumerate() {
        let line = line?;
        println!("Line {}: {}", i + 1, line);
    }

    Ok(())
}
```

## Reading Binary Data

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let bytes = fs::read("data.bin")?;
    println!("Read {} bytes", bytes.len());

    // Print first 10 bytes
    for (i, byte) in bytes.iter().take(10).enumerate() {
        println!("Byte {}: {:02x}", i, byte);
    }

    Ok(())
}
```

## Common File Operations

### **Check if File Exists**

```rust
use std::path::Path;

fn main() {
    let path = "hello.txt";

    if Path::new(path).exists() {
        println!("✅ File exists");
    } else {
        println!("❌ File not found");
    }
}
```

### **Get File Metadata**

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let metadata = fs::metadata("hello.txt")?;

    println!("File size: {} bytes", metadata.len());
    println!("Is file: {}", metadata.is_file());
    println!("Is directory: {}", metadata.is_dir());
    println!("Read only: {}", metadata.permissions().readonly());

    Ok(())
}
```

### **Read with Size Limit**

```rust
use std::fs;

fn read_small_file(path: &str, max_size: u64) -> Result<String, String> {
    let metadata = fs::metadata(path)
        .map_err(|e| format!("Cannot read metadata: {}", e))?;

    if metadata.len() > max_size {
        return Err(format!(
            "File too large: {} bytes (max: {})",
            metadata.len(),
            max_size
        ));
    }

    fs::read_to_string(path).map_err(|e| format!("Cannot read file: {}", e))
}

fn main() {
    match read_small_file("hello.txt", 1000) {
        Ok(content) => println!("Content: {}", content),
        Err(e) => println!("Error: {}", e),
    }
}
```

## Hands-On Practice

```bash
cargo new file_reader
cd file_reader
code .
```

### **Experiment 1: Basic File Reading**

```rust
use std::fs;

fn main() {
    // Create a test file
    fs::write("test.txt", "Hello, Rust!\nWelcome to file I/O.")
        .expect("Failed to create test file");

    // Read it back
    let content = fs::read_to_string("test.txt")
        .expect("Failed to read test file");

    println!("File contents:\n{}", content);
}
```

### **Experiment 2: Line Counter**

```rust
use std::error::Error;
use std::fs;

fn count_lines(path: &str) -> Result<usize, Box<dyn Error>> {
    let content = fs::read_to_string(path)?;
    let count = content.lines().count();
    Ok(count)
}

fn main() -> Result<(), Box<dyn Error>> {
    // Create test file with multiple lines
    fs::write("lines.txt", "Line 1\nLine 2\nLine 3\nLine 4\nLine 5")?;

    let count = count_lines("lines.txt")?;
    println!("File has {} lines", count);

    Ok(())
}
```

### **Experiment 3: Word Counter**

```rust
use std::error::Error;
use std::fs;

fn count_words(path: &str) -> Result<usize, Box<dyn Error>> {
    let content = fs::read_to_string(path)?;
    let count = content.split_whitespace().count();
    Ok(count)
}

fn count_chars(path: &str) -> Result<usize, Box<dyn Error>> {
    let content = fs::read_to_string(path)?;
    Ok(content.len())
}

fn main() -> Result<(), Box<dyn Error>> {
    let path = "sample.txt";

    // Create test file
    fs::write(
        path,
        "The quick brown fox jumps over the lazy dog.\nRust is awesome!",
    )?;

    let words = count_words(path)?;
    let chars = count_chars(path)?;
    let lines = fs::read_to_string(path)?.lines().count();

    println!("File statistics:");
    println!("  Lines: {}", lines);
    println!("  Words: {}", words);
    println!("  Characters: {}", chars);

    Ok(())
}
```

### **Experiment 4: Search in File**

```rust
use std::error::Error;
use std::fs;

fn search_in_file(path: &str, query: &str) -> Result<Vec<String>, Box<dyn Error>> {
    let content = fs::read_to_string(path)?;
    let matches: Vec<String> = content
        .lines()
        .filter(|line| line.contains(query))
        .map(|line| line.to_string())
        .collect();

    Ok(matches)
}

fn main() -> Result<(), Box<dyn Error>> {
    // Create test file
    fs::write(
        "data.txt",
        "Rust is great\nPython is popular\nRust is fast\nJava is old\nRust is safe",
    )?;

    let query = "Rust";
    let matches = search_in_file("data.txt", query)?;

    println!("Lines containing '{}':", query);
    for (i, line) in matches.iter().enumerate() {
        println!("{}. {}", i + 1, line);
    }

    println!("\nFound {} matching lines", matches.len());

    Ok(())
}
```

**Output:**
```
Lines containing 'Rust':
1. Rust is great
2. Rust is fast
3. Rust is safe

Found 3 matching lines
```

### **Challenge: Log File Analyzer**

Build a log file analyzer:

```rust
use std::error::Error;
use std::fs;

#[derive(Debug)]
struct LogStats {
    total_lines: usize,
    error_count: usize,
    warning_count: usize,
    info_count: usize,
}

fn analyze_log(path: &str) -> Result<LogStats, Box<dyn Error>> {
    let content = fs::read_to_string(path)?;
    let lines: Vec<&str> = content.lines().collect();

    let total_lines = lines.len();
    let error_count = lines.iter().filter(|line| line.contains("ERROR")).count();
    let warning_count = lines.iter().filter(|line| line.contains("WARN")).count();
    let info_count = lines.iter().filter(|line| line.contains("INFO")).count();

    Ok(LogStats {
        total_lines,
        error_count,
        warning_count,
        info_count,
    })
}

fn find_errors(path: &str) -> Result<Vec<String>, Box<dyn Error>> {
    let content = fs::read_to_string(path)?;
    let errors: Vec<String> = content
        .lines()
        .filter(|line| line.contains("ERROR"))
        .map(|line| line.to_string())
        .collect();

    Ok(errors)
}

fn main() -> Result<(), Box<dyn Error>> {
    // Create sample log file
    let log_content = "\
[2024-01-01 10:00:00] INFO: Application started
[2024-01-01 10:01:00] INFO: User logged in
[2024-01-01 10:02:00] WARN: High memory usage
[2024-01-01 10:03:00] ERROR: Database connection failed
[2024-01-01 10:04:00] INFO: Retrying connection
[2024-01-01 10:05:00] ERROR: Authentication failed for user john
[2024-01-01 10:06:00] WARN: Disk space low
[2024-01-01 10:07:00] INFO: Task completed successfully
[2024-01-01 10:08:00] ERROR: File not found: data.csv
[2024-01-01 10:09:00] INFO: Application shutdown
";

    fs::write("app.log", log_content)?;

    // Analyze log
    let stats = analyze_log("app.log")?;

    println!("╔═══════════════════════════════════╗");
    println!("║        Log File Analysis          ║");
    println!("╠═══════════════════════════════════╣");
    println!("║ Total lines:    {:17} ║", stats.total_lines);
    println!("║ INFO messages:  {:17} ║", stats.info_count);
    println!("║ WARN messages:  {:17} ║", stats.warning_count);
    println!("║ ERROR messages: {:17} ║", stats.error_count);
    println!("╚═══════════════════════════════════╝");

    // Show errors
    if stats.error_count > 0 {
        println!("\n🚨 Errors found:");
        let errors = find_errors("app.log")?;
        for (i, error) in errors.iter().enumerate() {
            println!("{}. {}", i + 1, error);
        }
    }

    Ok(())
}
```

**Output:**
```
╔═══════════════════════════════════╗
║        Log File Analysis          ║
╠═══════════════════════════════════╣
║ Total lines:                   10 ║
║ INFO messages:                  4 ║
║ WARN messages:                  2 ║
║ ERROR messages:                 3 ║
╚═══════════════════════════════════╝

🚨 Errors found:
1. [2024-01-01 10:03:00] ERROR: Database connection failed
2. [2024-01-01 10:05:00] ERROR: Authentication failed for user john
3. [2024-01-01 10:08:00] ERROR: File not found: data.csv
```

## Performance Considerations

### **When to Use read_to_string()**

```rust
// ✅ Good for small to medium files (< 100 MB)
let content = fs::read_to_string("config.json")?;
```

### **When to Use BufReader**

```rust
// ✅ Good for large files or when you need line-by-line processing
use std::fs::File;
use std::io::{BufRead, BufReader};

let file = File::open("huge_log.txt")?;
let reader = BufReader::new(file);

for line in reader.lines() {
    let line = line?;
    // Process each line without loading entire file
}
```

## Error Handling Best Practices

### **Specific Error Messages**

```rust
use std::fs;

fn read_config(path: &str) -> Result<String, String> {
    fs::read_to_string(path).map_err(|e| {
        format!("Failed to read config file '{}': {}", path, e)
    })
}

fn main() {
    match read_config("config.toml") {
        Ok(content) => println!("Config loaded"),
        Err(e) => eprintln!("Error: {}", e),
    }
}
```

### **Fallback to Defaults**

```rust
use std::fs;

fn load_or_default(path: &str, default: &str) -> String {
    fs::read_to_string(path).unwrap_or_else(|_| default.to_string())
}

fn main() {
    let config = load_or_default("config.txt", "default config");
    println!("Using config: {}", config);
}
```

## Key Takeaways

- ✅ `fs::read_to_string(path)` reads entire file as String
- ✅ `fs::read(path)` reads file as bytes (`Vec<u8>`)
- ✅ Use `BufReader` for large files or line-by-line reading
- ✅ Always handle errors when reading files
- ✅ Use `Path::new(path).exists()` to check file existence
- ✅ Use `fs::metadata()` to get file information
- ✅ The `?` operator makes error propagation clean
- ✅ Consider file size and memory when choosing read method
- ✅ Provide specific error messages for better debugging
- ✅ Test with missing files and invalid paths

**Next**: Writing data to files!

---

**Progress**: Module 7, Lesson 2 complete (40/60 lessons total)
