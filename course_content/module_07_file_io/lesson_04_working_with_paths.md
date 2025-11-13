# Module 7, Lesson 4: Navigating the File System — Working with Paths

## The Concept: Street Addresses

Imagine addresses:
- **Absolute**: "123 Main St, New York, NY" (complete address)
- **Relative**: "2 blocks north" (from current location)
- **Normalized**: Handle "going up" (../) and shortcuts

**Path** operations let you manipulate file locations safely and portably.

## The Path and PathBuf Types

```rust
use std::path::{Path, PathBuf};

fn main() {
    // &Path: borrowed path (like &str)
    let path = Path::new("data/files/document.txt");
    println!("Path: {:?}", path);

    // PathBuf: owned path (like String)
    let mut pathbuf = PathBuf::from("data");
    pathbuf.push("files");
    pathbuf.push("document.txt");
    println!("PathBuf: {:?}", pathbuf);
}
```

**Output:**
```
Path: "data/files/document.txt"
PathBuf: "data/files/document.txt"
```

## Path Components

```rust
use std::path::Path;

fn main() {
    let path = Path::new("data/logs/2024/app.log");

    // File name
    if let Some(name) = path.file_name() {
        println!("File name: {:?}", name);  // "app.log"
    }

    // Extension
    if let Some(ext) = path.extension() {
        println!("Extension: {:?}", ext);  // "log"
    }

    // File stem (name without extension)
    if let Some(stem) = path.file_stem() {
        println!("File stem: {:?}", stem);  // "app"
    }

    // Parent directory
    if let Some(parent) = path.parent() {
        println!("Parent: {:?}", parent);  // "data/logs/2024"
    }

    // Components
    println!("\nComponents:");
    for component in path.components() {
        println!("  {:?}", component);
    }
}
```

**Output:**
```
File name: "app.log"
Extension: "log"
File stem: "app"
Parent: "data/logs/2024"

Components:
  Normal("data")
  Normal("logs")
  Normal("2024")
  Normal("app.log")
```

## Building Paths

```rust
use std::path::PathBuf;

fn main() {
    let mut path = PathBuf::from("data");

    // Add components
    path.push("logs");
    path.push("2024");
    path.push("app.log");

    println!("Built path: {:?}", path);  // "data/logs/2024/app.log"

    // Remove last component
    path.pop();
    println!("After pop: {:?}", path);   // "data/logs/2024"

    // Replace filename
    path.push("system.log");
    println!("New file: {:?}", path);    // "data/logs/2024/system.log"
}
```

## Joining Paths

```rust
use std::path::Path;

fn main() {
    let base = Path::new("data/documents");
    let file = "report.pdf";

    let full_path = base.join(file);
    println!("Full path: {:?}", full_path);  // "data/documents/report.pdf"

    // Chaining joins
    let nested = Path::new("data")
        .join("projects")
        .join("rust")
        .join("main.rs");

    println!("Nested: {:?}", nested);  // "data/projects/rust/main.rs"
}
```

## Absolute vs Relative Paths

```rust
use std::env;
use std::path::Path;

fn main() -> std::io::Result<()> {
    let relative = Path::new("data/file.txt");
    let absolute = relative.canonicalize()?;  // Converts to absolute

    println!("Relative: {:?}", relative);
    println!("Absolute: {:?}", absolute);

    // Check if path is absolute
    println!("Is absolute: {}", absolute.is_absolute());
    println!("Is relative: {}", relative.is_relative());

    Ok(())
}
```

## Checking Path Properties

```rust
use std::path::Path;

fn main() {
    let path = Path::new("src/main.rs");

    println!("Exists: {}", path.exists());
    println!("Is file: {}", path.is_file());
    println!("Is directory: {}", path.is_dir());
    println!("Is absolute: {}", path.is_absolute());
    println!("Is relative: {}", path.is_relative());
}
```

## Cross-Platform Path Handling

```rust
use std::path::{Path, MAIN_SEPARATOR};

fn main() {
    println!("Path separator: {}", MAIN_SEPARATOR);
    // Windows: \
    // Unix: /

    // PathBuf handles platform differences automatically
    let path = Path::new("data").join("files").join("doc.txt");

    println!("Path: {:?}", path);
    // Windows: "data\\files\\doc.txt"
    // Unix: "data/files/doc.txt"
}
```

## Hands-On Practice

```bash
cargo new path_practice
cd path_practice
code .
```

### **Experiment 1: Path Dissection**

```rust
use std::path::Path;

fn analyze_path(path_str: &str) {
    let path = Path::new(path_str);

    println!("╔═══════════════════════════════════╗");
    println!("║      Path Analysis                ║");
    println!("╠═══════════════════════════════════╣");
    println!("║ Full path: {:19} ║", path_str);

    if let Some(name) = path.file_name().and_then(|n| n.to_str()) {
        println!("║ File name: {:19} ║", name);
    }

    if let Some(ext) = path.extension().and_then(|e| e.to_str()) {
        println!("║ Extension: {:19} ║", ext);
    }

    if let Some(stem) = path.file_stem().and_then(|s| s.to_str()) {
        println!("║ File stem: {:19} ║", stem);
    }

    if let Some(parent) = path.parent() {
        println!("║ Parent:    {:19} ║", parent.to_str().unwrap_or(""));
    }

    println!("╚═══════════════════════════════════╝");
}

fn main() {
    analyze_path("data/logs/2024/app.log");
    println!();
    analyze_path("documents/report.pdf");
    println!();
    analyze_path("src/main.rs");
}
```

### **Experiment 2: Path Builder**

```rust
use std::path::PathBuf;

fn build_log_path(date: &str, level: &str) -> PathBuf {
    PathBuf::from("logs")
        .join(date)
        .join(format!("{}.log", level))
}

fn main() {
    let error_log = build_log_path("2024-01-15", "error");
    let info_log = build_log_path("2024-01-15", "info");
    let debug_log = build_log_path("2024-01-16", "debug");

    println!("Error log: {:?}", error_log);
    println!("Info log: {:?}", info_log);
    println!("Debug log: {:?}", debug_log);
}
```

**Output:**
```
Error log: "logs/2024-01-15/error.log"
Info log: "logs/2024-01-15/info.log"
Debug log: "logs/2024-01-16/debug.log"
```

### **Experiment 3: File Organizer**

```rust
use std::fs;
use std::path::Path;

fn organize_file(file_path: &str) -> Result<String, String> {
    let path = Path::new(file_path);

    let extension = path
        .extension()
        .and_then(|ext| ext.to_str())
        .ok_or("No extension found")?;

    let category = match extension {
        "txt" | "md" | "doc" => "documents",
        "jpg" | "png" | "gif" => "images",
        "mp3" | "wav" | "flac" => "audio",
        "mp4" | "avi" | "mkv" => "video",
        "rs" | "py" | "js" => "code",
        _ => "other",
    };

    let filename = path
        .file_name()
        .and_then(|name| name.to_str())
        .ok_or("Invalid filename")?;

    let target = Path::new("organized")
        .join(category)
        .join(filename);

    Ok(format!("{} -> {:?}", file_path, target))
}

fn main() {
    let files = vec![
        "report.txt",
        "photo.jpg",
        "song.mp3",
        "main.rs",
        "movie.mp4",
        "data.csv",
    ];

    println!("File Organization Plan:\n");

    for file in files {
        match organize_file(file) {
            Ok(plan) => println!("✅ {}", plan),
            Err(e) => println!("❌ {}: {}", file, e),
        }
    }
}
```

**Output:**
```
File Organization Plan:

✅ report.txt -> "organized/documents/report.txt"
✅ photo.jpg -> "organized/images/photo.jpg"
✅ song.mp3 -> "organized/audio/song.mp3"
✅ main.rs -> "organized/code/main.rs"
✅ movie.mp4 -> "organized/video/movie.mp4"
✅ data.csv -> "organized/other/data.csv"
```

### **Challenge: Recursive Directory Walker**

```rust
use std::fs;
use std::path::Path;

fn walk_directory(dir: &Path, indent: usize) -> std::io::Result<()> {
    if !dir.is_dir() {
        return Ok(());
    }

    let entries = fs::read_dir(dir)?;

    for entry in entries {
        let entry = entry?;
        let path = entry.path();
        let name = path
            .file_name()
            .and_then(|n| n.to_str())
            .unwrap_or("?");

        // Print with indentation
        let prefix = "  ".repeat(indent);

        if path.is_dir() {
            println!("{}📁 {}/", prefix, name);
            walk_directory(&path, indent + 1)?;
        } else {
            let size = entry.metadata()?.len();
            println!("{}📄 {} ({} bytes)", prefix, name, size);
        }
    }

    Ok(())
}

fn main() -> std::io::Result<()> {
    // Create test directory structure
    fs::create_dir_all("test_dir/subdir1")?;
    fs::create_dir_all("test_dir/subdir2")?;
    fs::write("test_dir/file1.txt", "Hello")?;
    fs::write("test_dir/subdir1/file2.txt", "World")?;
    fs::write("test_dir/subdir2/file3.txt", "Rust")?;

    println!("Directory structure:\n");
    walk_directory(Path::new("test_dir"), 0)?;

    Ok(())
}
```

**Output:**
```
Directory structure:

📁 subdir1/
  📄 file2.txt (5 bytes)
📁 subdir2/
  📄 file3.txt (4 bytes)
📄 file1.txt (5 bytes)
```

## Working with Current Directory

```rust
use std::env;

fn main() -> std::io::Result<()> {
    // Get current directory
    let current = env::current_dir()?;
    println!("Current directory: {:?}", current);

    // Change current directory
    env::set_current_dir("src")?;
    let new_current = env::current_dir()?;
    println!("New directory: {:?}", new_current);

    // Change back
    env::set_current_dir("..")?;
    let restored = env::current_dir()?;
    println!("Restored directory: {:?}", restored);

    Ok(())
}
```

## Path Normalization

```rust
use std::path::{Path, PathBuf};

fn normalize_path(path: &str) -> PathBuf {
    let mut components = Vec::new();

    for component in Path::new(path).components() {
        match component {
            std::path::Component::ParentDir => {
                components.pop();  // Go up one level
            }
            std::path::Component::CurDir => {
                // Skip "." (current directory)
            }
            _ => {
                components.push(component);
            }
        }
    }

    components.iter().collect()
}

fn main() {
    let paths = vec![
        "data/./files/document.txt",
        "data/../data/files/document.txt",
        "./data/files/../docs/report.pdf",
    ];

    for path in paths {
        let normalized = normalize_path(path);
        println!("{:35} => {:?}", path, normalized);
    }
}
```

**Output:**
```
data/./files/document.txt           => "data/files/document.txt"
data/../data/files/document.txt     => "data/files/document.txt"
./data/files/../docs/report.pdf     => "data/docs/report.pdf"
```

## Common Patterns

### **Pattern 1: Safe Filename**

```rust
fn sanitize_filename(name: &str) -> String {
    name.chars()
        .map(|c| match c {
            '/' | '\\' | ':' | '*' | '?' | '"' | '<' | '>' | '|' => '_',
            c => c,
        })
        .collect()
}

fn main() {
    let unsafe_name = "Report: Q1/2024 (Final).pdf";
    let safe_name = sanitize_filename(unsafe_name);
    println!("Safe filename: {}", safe_name);
    // "Report_ Q1_2024 (Final).pdf"
}
```

### **Pattern 2: Backup Path Generator**

```rust
use std::path::{Path, PathBuf};

fn create_backup_path(original: &Path) -> PathBuf {
    let mut backup = original.to_path_buf();

    if let Some(stem) = original.file_stem() {
        if let Some(ext) = original.extension() {
            let backup_name = format!(
                "{}.backup.{}",
                stem.to_str().unwrap(),
                ext.to_str().unwrap()
            );
            backup.set_file_name(backup_name);
        }
    }

    backup
}

fn main() {
    let original = Path::new("data/important.txt");
    let backup = create_backup_path(original);
    println!("Original: {:?}", original);
    println!("Backup:   {:?}", backup);
}
```

### **Pattern 3: Find Files by Extension**

```rust
use std::fs;
use std::path::Path;

fn find_files_by_extension(dir: &Path, ext: &str) -> std::io::Result<Vec<String>> {
    let mut results = Vec::new();

    if !dir.is_dir() {
        return Ok(results);
    }

    for entry in fs::read_dir(dir)? {
        let entry = entry?;
        let path = entry.path();

        if path.is_file() {
            if let Some(file_ext) = path.extension() {
                if file_ext == ext {
                    if let Some(name) = path.to_str() {
                        results.push(name.to_string());
                    }
                }
            }
        }
    }

    Ok(results)
}

fn main() -> std::io::Result<()> {
    // Create test files
    fs::write("test1.rs", "// Rust file 1")?;
    fs::write("test2.rs", "// Rust file 2")?;
    fs::write("readme.md", "# Readme")?;

    let rust_files = find_files_by_extension(Path::new("."), "rs")?;

    println!("Rust files:");
    for file in rust_files {
        println!("  - {}", file);
    }

    Ok(())
}
```

## Key Takeaways

- ✅ `Path` is for borrowed paths (like &str), `PathBuf` for owned paths (like String)
- ✅ Use `.join()` to combine path components safely
- ✅ `.file_name()`, `.extension()`, `.parent()` extract path parts
- ✅ `.exists()`, `.is_file()`, `.is_dir()` check path properties
- ✅ Paths are cross-platform - Rust handles separators automatically
- ✅ Use `.canonicalize()` to get absolute paths
- ✅ `.push()` adds components to PathBuf, `.pop()` removes last
- ✅ Always handle path operations that return `Option` safely
- ✅ Sanitize user-provided filenames to prevent security issues
- ✅ Use `Path::new()` to create paths from strings

**Next**: Practice project bringing all file I/O concepts together!

---

**Progress**: Module 7, Lesson 4 complete (42/60 lessons total)
