# Module 7, Lesson 5: Practice Project — Building a File Management Tool

## Project Overview

Build a comprehensive command-line file management tool that demonstrates all file I/O techniques!

**What we're building:**
- File and directory operations
- Search and filter capabilities
- File content analysis
- Batch operations
- Complete CLI interface

**Skills practiced:**
- ✅ Command-line argument parsing
- ✅ Reading and writing files
- ✅ Path manipulation
- ✅ Directory traversal
- ✅ Error handling
- ✅ All file I/O concepts working together

## The Complete Project

### **Step 1: Create the Project**

```bash
cargo new filetool
cd filetool
code .
```

### **Step 2: Add Dependencies to Cargo.toml**

```toml
[package]
name = "filetool"
version = "0.1.0"
edition = "2021"

[dependencies]
clap = { version = "4.5", features = ["derive"] }
```

### **Step 3: Build the Tool (Complete Code)**

```rust
use clap::{Parser, Subcommand};
use std::error::Error;
use std::fs;
use std::path::{Path, PathBuf};

// ===== CLI Definition =====

#[derive(Parser)]
#[command(name = "filetool")]
#[command(about = "A comprehensive file management tool", long_about = None)]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// List files in a directory
    List {
        /// Directory to list (default: current directory)
        #[arg(default_value = ".")]
        path: String,

        /// Show file sizes
        #[arg(short, long)]
        size: bool,
    },

    /// Search for files by name
    Search {
        /// Directory to search in
        path: String,

        /// Search pattern
        pattern: String,
    },

    /// Count words in a text file
    Count {
        /// File to analyze
        file: String,
    },

    /// Copy a file
    Copy {
        /// Source file
        source: String,

        /// Destination file
        dest: String,
    },

    /// Find and replace text in a file
    Replace {
        /// File to modify
        file: String,

        /// Text to find
        find: String,

        /// Replacement text
        replace: String,
    },

    /// Create a backup of a file
    Backup {
        /// File to backup
        file: String,
    },

    /// Get file information
    Info {
        /// File to analyze
        file: String,
    },
}

// ===== Implementation =====

fn list_files(path: &str, show_size: bool) -> Result<(), Box<dyn Error>> {
    let dir_path = Path::new(path);

    if !dir_path.exists() {
        return Err(format!("Directory '{}' not found", path).into());
    }

    if !dir_path.is_dir() {
        return Err(format!("'{}' is not a directory", path).into());
    }

    println!("📂 Contents of {}:\n", path);

    let mut entries: Vec<_> = fs::read_dir(dir_path)?.collect();
    entries.sort_by_key(|e| e.as_ref().ok().map(|e| e.path()));

    for entry in entries {
        let entry = entry?;
        let path = entry.path();
        let metadata = entry.metadata()?;

        let name = path
            .file_name()
            .and_then(|n| n.to_str())
            .unwrap_or("?");

        if path.is_dir() {
            if show_size {
                println!("📁 {}/", name);
            } else {
                println!("📁 {}/", name);
            }
        } else {
            if show_size {
                let size = metadata.len();
                println!("📄 {} ({} bytes)", name, size);
            } else {
                println!("📄 {}", name);
            }
        }
    }

    Ok(())
}

fn search_files(dir: &str, pattern: &str) -> Result<(), Box<dyn Error>> {
    let search_path = Path::new(dir);

    if !search_path.exists() {
        return Err(format!("Directory '{}' not found", dir).into());
    }

    println!("🔍 Searching for '{}' in {}...\n", pattern, dir);

    let mut matches = Vec::new();
    search_recursive(search_path, pattern, &mut matches)?;

    if matches.is_empty() {
        println!("No files found matching '{}'", pattern);
    } else {
        println!("Found {} matching file(s):\n", matches.len());
        for (i, file) in matches.iter().enumerate() {
            println!("{}. {}", i + 1, file);
        }
    }

    Ok(())
}

fn search_recursive(
    dir: &Path,
    pattern: &str,
    matches: &mut Vec<String>,
) -> Result<(), Box<dyn Error>> {
    for entry in fs::read_dir(dir)? {
        let entry = entry?;
        let path = entry.path();

        if path.is_dir() {
            search_recursive(&path, pattern, matches)?;
        } else if path.is_file() {
            if let Some(name) = path.file_name().and_then(|n| n.to_str()) {
                if name.contains(pattern) {
                    if let Some(path_str) = path.to_str() {
                        matches.push(path_str.to_string());
                    }
                }
            }
        }
    }

    Ok(())
}

fn count_file(file: &str) -> Result<(), Box<dyn Error>> {
    let path = Path::new(file);

    if !path.exists() {
        return Err(format!("File '{}' not found", file).into());
    }

    let content = fs::read_to_string(path)?;

    let lines = content.lines().count();
    let words = content.split_whitespace().count();
    let chars = content.len();
    let bytes = path.metadata()?.len();

    println!("╔═══════════════════════════════════╗");
    println!("║      File Statistics              ║");
    println!("╠═══════════════════════════════════╣");
    println!("║ File:       {:19} ║", file);
    println!("║ Lines:      {:19} ║", lines);
    println!("║ Words:      {:19} ║", words);
    println!("║ Characters: {:19} ║", chars);
    println!("║ Bytes:      {:19} ║", bytes);
    println!("╚═══════════════════════════════════╝");

    Ok(())
}

fn copy_file(source: &str, dest: &str) -> Result<(), Box<dyn Error>> {
    let src_path = Path::new(source);
    let dst_path = Path::new(dest);

    if !src_path.exists() {
        return Err(format!("Source file '{}' not found", source).into());
    }

    if dst_path.exists() {
        return Err(format!("Destination file '{}' already exists", dest).into());
    }

    // Create parent directories if needed
    if let Some(parent) = dst_path.parent() {
        fs::create_dir_all(parent)?;
    }

    fs::copy(src_path, dst_path)?;

    println!("✅ Copied '{}' to '{}'", source, dest);

    Ok(())
}

fn replace_in_file(file: &str, find: &str, replace: &str) -> Result<(), Box<dyn Error>> {
    let path = Path::new(file);

    if !path.exists() {
        return Err(format!("File '{}' not found", file).into());
    }

    let content = fs::read_to_string(path)?;
    let occurrences = content.matches(find).count();

    if occurrences == 0 {
        println!("⚠️  No occurrences of '{}' found in '{}'", find, file);
        return Ok(());
    }

    let new_content = content.replace(find, replace);
    fs::write(path, new_content)?;

    println!(
        "✅ Replaced {} occurrence(s) of '{}' with '{}' in '{}'",
        occurrences, find, replace, file
    );

    Ok(())
}

fn backup_file(file: &str) -> Result<(), Box<dyn Error>> {
    let path = Path::new(file);

    if !path.exists() {
        return Err(format!("File '{}' not found", file).into());
    }

    let backup_path = create_backup_path(path);
    fs::copy(path, &backup_path)?;

    println!(
        "✅ Created backup: {}",
        backup_path.to_str().unwrap_or("?")
    );

    Ok(())
}

fn create_backup_path(original: &Path) -> PathBuf {
    let mut backup = original.to_path_buf();

    if let Some(stem) = original.file_stem() {
        let timestamp = std::time::SystemTime::now()
            .duration_since(std::time::UNIX_EPOCH)
            .unwrap()
            .as_secs();

        let backup_name = if let Some(ext) = original.extension() {
            format!(
                "{}.backup_{}.{}",
                stem.to_str().unwrap(),
                timestamp,
                ext.to_str().unwrap()
            )
        } else {
            format!("{}.backup_{}", stem.to_str().unwrap(), timestamp)
        };

        backup.set_file_name(backup_name);
    }

    backup
}

fn file_info(file: &str) -> Result<(), Box<dyn Error>> {
    let path = Path::new(file);

    if !path.exists() {
        return Err(format!("File '{}' not found", file).into());
    }

    let metadata = path.metadata()?;
    let file_type = if metadata.is_file() {
        "File"
    } else if metadata.is_dir() {
        "Directory"
    } else {
        "Other"
    };

    let size = metadata.len();
    let readonly = metadata.permissions().readonly();

    let extension = path
        .extension()
        .and_then(|e| e.to_str())
        .unwrap_or("(none)");

    let parent = path
        .parent()
        .and_then(|p| p.to_str())
        .unwrap_or("(none)");

    println!("╔═══════════════════════════════════╗");
    println!("║      File Information             ║");
    println!("╠═══════════════════════════════════╣");
    println!("║ Path:      {:20} ║", file);
    println!("║ Type:      {:20} ║", file_type);
    println!("║ Size:      {:17} bytes ║", size);
    println!("║ Extension: {:20} ║", extension);
    println!("║ Parent:    {:20} ║", parent);
    println!("║ Read-only: {:20} ║", readonly);
    println!("╚═══════════════════════════════════╝");

    Ok(())
}

// ===== Main Function =====

fn main() {
    let cli = Cli::parse();

    let result = match cli.command {
        Commands::List { path, size } => list_files(&path, size),
        Commands::Search { path, pattern } => search_files(&path, &pattern),
        Commands::Count { file } => count_file(&file),
        Commands::Copy { source, dest } => copy_file(&source, &dest),
        Commands::Replace {
            file,
            find,
            replace,
        } => replace_in_file(&file, &find, &replace),
        Commands::Backup { file } => backup_file(&file),
        Commands::Info { file } => file_info(&file),
    };

    if let Err(e) = result {
        eprintln!("❌ Error: {}", e);
        std::process::exit(1);
    }
}
```

### **Step 4: Build and Test**

```bash
cargo build --release
```

## Usage Examples

### **List Files**

```bash
cargo run -- list .
cargo run -- list src --size
```

**Output:**
```
📂 Contents of src:

📄 main.rs (5234 bytes)
```

### **Search for Files**

```bash
# Create test files first
mkdir -p test_data
echo "Hello" > test_data/hello.txt
echo "World" > test_data/world.txt
echo "Test" > test_data/test.md

cargo run -- search test_data txt
```

**Output:**
```
🔍 Searching for 'txt' in test_data...

Found 2 matching file(s):

1. test_data/hello.txt
2. test_data/world.txt
```

### **Count Words**

```bash
echo "The quick brown fox jumps over the lazy dog" > sample.txt
cargo run -- count sample.txt
```

**Output:**
```
╔═══════════════════════════════════╗
║      File Statistics              ║
╠═══════════════════════════════════╣
║ File:       sample.txt            ║
║ Lines:      1                     ║
║ Words:      9                     ║
║ Characters: 43                    ║
║ Bytes:      43                    ║
╚═══════════════════════════════════╝
```

### **Copy File**

```bash
cargo run -- copy sample.txt copy_of_sample.txt
```

**Output:**
```
✅ Copied 'sample.txt' to 'copy_of_sample.txt'
```

### **Find and Replace**

```bash
cargo run -- replace sample.txt "fox" "cat"
```

**Output:**
```
✅ Replaced 1 occurrence(s) of 'fox' with 'cat' in 'sample.txt'
```

### **Create Backup**

```bash
cargo run -- backup sample.txt
```

**Output:**
```
✅ Created backup: sample.backup_1704153600.txt
```

### **Get File Info**

```bash
cargo run -- info sample.txt
```

**Output:**
```
╔═══════════════════════════════════╗
║      File Information             ║
╠═══════════════════════════════════╣
║ Path:      sample.txt             ║
║ Type:      File                   ║
║ Size:      43 bytes               ║
║ Extension: txt                    ║
║ Parent:    .                      ║
║ Read-only: false                  ║
╚═══════════════════════════════════╝
```

## Understanding the Code

### **Command-Line Interface with clap**

```rust
#[derive(Parser)]
#[command(name = "filetool")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}
```

**Why**: Professional CLI with automatic help generation and parsing.

### **Recursive Directory Search**

```rust
fn search_recursive(dir: &Path, pattern: &str, matches: &mut Vec<String>) -> Result<(), Box<dyn Error>> {
    for entry in fs::read_dir(dir)? {
        let path = entry.path();
        if path.is_dir() {
            search_recursive(&path, pattern, matches)?;  // Recurse
        }
        // ...
    }
    Ok(())
}
```

**Why**: Searches all subdirectories systematically.

### **Safe File Operations**

```rust
if !src_path.exists() {
    return Err(format!("Source file '{}' not found", source).into());
}

if dst_path.exists() {
    return Err(format!("Destination file '{}' already exists", dest).into());
}
```

**Why**: Prevents accidental data loss and provides clear error messages.

### **Timestamp-Based Backups**

```rust
let timestamp = std::time::SystemTime::now()
    .duration_since(std::time::UNIX_EPOCH)
    .unwrap()
    .as_secs();

let backup_name = format!("{}.backup_{}.{}", stem, timestamp, ext);
```

**Why**: Unique backup filenames prevent overwriting previous backups.

## Challenges and Extensions

### **Challenge 1: Add Delete Command**

```rust
Commands::Delete {
    /// File to delete
    file: String,

    /// Skip confirmation
    #[arg(short, long)]
    force: bool,
},

fn delete_file(file: &str, force: bool) -> Result<(), Box<dyn Error>> {
    let path = Path::new(file);

    if !path.exists() {
        return Err(format!("File '{}' not found", file).into());
    }

    if !force {
        use std::io::{self, Write};
        print!("Delete '{}'? (y/N): ", file);
        io::stdout().flush()?;

        let mut input = String::new();
        io::stdin().read_line(&mut input)?;

        if input.trim().to_lowercase() != "y" {
            println!("Cancelled");
            return Ok(());
        }
    }

    fs::remove_file(path)?;
    println!("✅ Deleted '{}'", file);

    Ok(())
}
```

### **Challenge 2: Add Move Command**

```rust
Commands::Move {
    /// Source file
    source: String,

    /// Destination
    dest: String,
},

fn move_file(source: &str, dest: &str) -> Result<(), Box<dyn Error>> {
    let src_path = Path::new(source);
    let dst_path = Path::new(dest);

    if !src_path.exists() {
        return Err(format!("Source '{}' not found", source).into());
    }

    fs::rename(src_path, dst_path)?;
    println!("✅ Moved '{}' to '{}'", source, dest);

    Ok(())
}
```

### **Challenge 3: Add File Comparison**

```rust
Commands::Compare {
    /// First file
    file1: String,

    /// Second file
    file2: String,
},

fn compare_files(file1: &str, file2: &str) -> Result<(), Box<dyn Error>> {
    let content1 = fs::read_to_string(file1)?;
    let content2 = fs::read_to_string(file2)?;

    if content1 == content2 {
        println!("✅ Files are identical");
    } else {
        println!("❌ Files are different");

        let lines1: Vec<&str> = content1.lines().collect();
        let lines2: Vec<&str> = content2.lines().collect();

        for (i, (line1, line2)) in lines1.iter().zip(lines2.iter()).enumerate() {
            if line1 != line2 {
                println!("Difference at line {}:", i + 1);
                println!("  File 1: {}", line1);
                println!("  File 2: {}", line2);
            }
        }
    }

    Ok(())
}
```

### **Challenge 4: Add File Statistics Summary**

```rust
Commands::Stats {
    /// Directory to analyze
    #[arg(default_value = ".")]
    path: String,
},

fn directory_stats(path: &str) -> Result<(), Box<dyn Error>> {
    let mut total_files = 0;
    let mut total_dirs = 0;
    let mut total_size = 0u64;

    for entry in fs::read_dir(path)? {
        let entry = entry?;
        let metadata = entry.metadata()?;

        if metadata.is_file() {
            total_files += 1;
            total_size += metadata.len();
        } else if metadata.is_dir() {
            total_dirs += 1;
        }
    }

    println!("╔═══════════════════════════════════╗");
    println!("║    Directory Statistics           ║");
    println!("╠═══════════════════════════════════╣");
    println!("║ Files:       {:17} ║", total_files);
    println!("║ Directories: {:17} ║", total_dirs);
    println!("║ Total size:  {:14} bytes ║", total_size);
    println!("╚═══════════════════════════════════╝");

    Ok(())
}
```

## Key Takeaways

- ✅ Command-line tools combine multiple file I/O operations
- ✅ The `clap` crate makes building professional CLIs easy
- ✅ Always validate inputs and handle errors gracefully
- ✅ Recursive functions enable directory tree traversal
- ✅ Path manipulation ensures cross-platform compatibility
- ✅ Safety checks prevent accidental data loss
- ✅ Timestamps create unique backup filenames
- ✅ Comprehensive error messages improve user experience
- ✅ Modular functions make code maintainable and testable
- ✅ All file I/O concepts work together in real applications

---

## ✅ Module 7 Complete!

You've mastered Rust file I/O:
- ✅ Command-line argument parsing
- ✅ Reading files (whole and line-by-line)
- ✅ Writing files (create and append)
- ✅ Path manipulation and navigation
- ✅ Built a complete file management tool!

**Next: Module 8 — Traits & Generics**

Total Progress: 43 lessons complete (~72%)

---

[← Back to Module 7](README.md) | [Continue to Module 8 →](../module_08_traits_generics/)
