# Module 7, Lesson 3: Saving Data to Disk — Writing Files

## The Concept: Writing in a Notebook

Imagine a notebook:
- **Write on blank page**: Create new content
- **Add to existing page**: Append more content
- **Erase and rewrite**: Overwrite existing content

**File writing** lets your program save data persistently to disk.

## The Simplest Way: write()

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::write("output.txt", "Hello, file system!")?;
    println!("✅ File written successfully!");
    Ok(())
}
```

**This creates or overwrites the file!**

## Writing Multiple Times

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // First write (creates file)
    fs::write("log.txt", "Application started\n")?;

    // Second write (OVERWRITES file)
    fs::write("log.txt", "Application running\n")?;

    // Third write (OVERWRITES again)
    fs::write("log.txt", "Application stopped\n")?;

    // Only last message remains!
    let content = fs::read_to_string("log.txt")?;
    println!("File contains:\n{}", content);

    Ok(())
}
```

**Output:**
```
File contains:
Application stopped
```

## Appending to Files

To add content without erasing, use `OpenOptions`:

```rust
use std::fs::OpenOptions;
use std::io::Write;

fn main() -> std::io::Result<()> {
    // Create initial file
    std::fs::write("log.txt", "Application started\n")?;

    // Append more lines
    let mut file = OpenOptions::new()
        .append(true)
        .open("log.txt")?;

    writeln!(file, "User logged in")?;
    writeln!(file, "Task completed")?;
    writeln!(file, "Application stopped")?;

    println!("✅ Log entries appended!");

    // Read back
    let content = std::fs::read_to_string("log.txt")?;
    println!("\nLog file:\n{}", content);

    Ok(())
}
```

**Output:**
```
✅ Log entries appended!

Log file:
Application started
User logged in
Task completed
Application stopped
```

## Creating Directories

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // Create directory
    fs::create_dir("data")?;
    println!("✅ Directory created");

    // Create nested directories
    fs::create_dir_all("data/logs/2024")?;
    println!("✅ Nested directories created");

    // Write file in directory
    fs::write("data/logs/2024/app.log", "Log entry 1\n")?;
    println!("✅ File written to directory");

    Ok(())
}
```

## Safe File Writing

Check before overwriting:

```rust
use std::fs;
use std::path::Path;

fn safe_write(path: &str, content: &str) -> Result<(), String> {
    if Path::new(path).exists() {
        return Err(format!("File '{}' already exists!", path));
    }

    fs::write(path, content).map_err(|e| format!("Write error: {}", e))?;

    Ok(())
}

fn main() {
    match safe_write("important.txt", "Important data") {
        Ok(_) => println!("✅ File created"),
        Err(e) => println!("❌ Error: {}", e),
    }

    // Try again
    match safe_write("important.txt", "More data") {
        Ok(_) => println!("✅ File created"),
        Err(e) => println!("❌ Error: {}", e),  // Will error
    }
}
```

## Writing Different Data Types

### **Writing Numbers**

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let numbers = vec![1, 2, 3, 4, 5];

    // Convert to string
    let content: String = numbers
        .iter()
        .map(|n| n.to_string())
        .collect::<Vec<String>>()
        .join("\n");

    fs::write("numbers.txt", content)?;
    println!("✅ Numbers written");

    Ok(())
}
```

### **Writing CSV Data**

```rust
use std::fs;

struct Person {
    name: String,
    age: u32,
    city: String,
}

fn main() -> std::io::Result<()> {
    let people = vec![
        Person {
            name: String::from("Alice"),
            age: 30,
            city: String::from("New York"),
        },
        Person {
            name: String::from("Bob"),
            age: 25,
            city: String::from("San Francisco"),
        },
        Person {
            name: String::from("Charlie"),
            age: 35,
            city: String::from("Chicago"),
        },
    ];

    // Create CSV content
    let mut csv = String::from("name,age,city\n");

    for person in people {
        csv.push_str(&format!("{},{},{}\n", person.name, person.age, person.city));
    }

    fs::write("people.csv", csv)?;
    println!("✅ CSV file created");

    Ok(())
}
```

### **Writing Binary Data**

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let data: Vec<u8> = vec![0x48, 0x65, 0x6C, 0x6C, 0x6F];  // "Hello" in bytes

    fs::write("data.bin", data)?;
    println!("✅ Binary file written");

    // Read back
    let bytes = fs::read("data.bin")?;
    let text = String::from_utf8_lossy(&bytes);
    println!("Binary content as text: {}", text);

    Ok(())
}
```

## Hands-On Practice

```bash
cargo new file_writer
cd file_writer
code .
```

### **Experiment 1: Basic Writing**

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // Write
    fs::write("greeting.txt", "Hello, World!\n")?;
    fs::write("greeting.txt", "Welcome to Rust!\n")?;  // Overwrites

    // Read back
    let content = fs::read_to_string("greeting.txt")?;
    println!("File contains: {}", content);

    Ok(())
}
```

### **Experiment 2: Append Logger**

```rust
use std::fs::OpenOptions;
use std::io::Write;

fn log_message(message: &str) -> std::io::Result<()> {
    let mut file = OpenOptions::new()
        .create(true)
        .append(true)
        .open("app.log")?;

    let timestamp = chrono::Local::now().format("%Y-%m-%d %H:%M:%S");
    writeln!(file, "[{}] {}", timestamp, message)?;

    Ok(())
}

fn main() -> std::io::Result<()> {
    log_message("Application started")?;
    log_message("Processing data")?;
    log_message("Task completed")?;
    log_message("Application stopped")?;

    println!("✅ Log entries written");

    // Display log
    let log = std::fs::read_to_string("app.log")?;
    println!("\n📋 Log file:\n{}", log);

    Ok(())
}
```

**Note**: Add chrono to Cargo.toml or use simple strings:
```rust
fn log_message(message: &str) -> std::io::Result<()> {
    let mut file = OpenOptions::new()
        .create(true)
        .append(true)
        .open("app.log")?;

    writeln!(file, "{}", message)?;
    Ok(())
}
```

### **Experiment 3: Configuration Manager**

```rust
use std::fs;
use std::path::Path;

fn save_config(key: &str, value: &str) -> std::io::Result<()> {
    let config_path = "config.txt";
    let mut content = String::new();

    // Read existing config if it exists
    if Path::new(config_path).exists() {
        content = fs::read_to_string(config_path)?;
    }

    // Check if key exists
    let lines: Vec<&str> = content.lines().collect();
    let mut found = false;
    let mut new_content = String::new();

    for line in lines {
        if line.starts_with(&format!("{}=", key)) {
            new_content.push_str(&format!("{}={}\n", key, value));
            found = true;
        } else {
            new_content.push_str(line);
            new_content.push('\n');
        }
    }

    // Add new key if not found
    if !found {
        new_content.push_str(&format!("{}={}\n", key, value));
    }

    fs::write(config_path, new_content)?;
    Ok(())
}

fn load_config(key: &str) -> std::io::Result<Option<String>> {
    let content = fs::read_to_string("config.txt")?;

    for line in content.lines() {
        if line.starts_with(&format!("{}=", key)) {
            let value = line.split('=').nth(1).unwrap_or("").to_string();
            return Ok(Some(value));
        }
    }

    Ok(None)
}

fn main() -> std::io::Result<()> {
    // Save settings
    save_config("theme", "dark")?;
    save_config("language", "en")?;
    save_config("font_size", "14")?;

    println!("✅ Configuration saved");

    // Load settings
    if let Some(theme) = load_config("theme")? {
        println!("Theme: {}", theme);
    }

    if let Some(lang) = load_config("language")? {
        println!("Language: {}", lang);
    }

    // Update setting
    save_config("theme", "light")?;
    println!("✅ Theme updated");

    // Display config file
    let config = fs::read_to_string("config.txt")?;
    println!("\n⚙️  Config file:\n{}", config);

    Ok(())
}
```

### **Challenge: Note-Taking App**

```rust
use std::fs::{self, OpenOptions};
use std::io::Write;
use std::path::Path;

struct Note {
    title: String,
    content: String,
}

impl Note {
    fn new(title: &str, content: &str) -> Self {
        Note {
            title: title.to_string(),
            content: content.to_string(),
        }
    }

    fn save(&self) -> std::io::Result<()> {
        // Create notes directory if it doesn't exist
        fs::create_dir_all("notes")?;

        // Create filename from title
        let filename = format!("notes/{}.txt", self.title.replace(' ', "_"));

        // Write note
        let content = format!("Title: {}\n\n{}", self.title, self.content);
        fs::write(&filename, content)?;

        println!("✅ Note '{}' saved", self.title);
        Ok(())
    }

    fn load(title: &str) -> std::io::Result<Option<Note>> {
        let filename = format!("notes/{}.txt", title.replace(' ', "_"));

        if !Path::new(&filename).exists() {
            return Ok(None);
        }

        let content = fs::read_to_string(&filename)?;
        let lines: Vec<&str> = content.lines().collect();

        if lines.is_empty() {
            return Ok(None);
        }

        // Skip "Title: " line and get rest
        let note_content = lines[2..].join("\n");

        Ok(Some(Note {
            title: title.to_string(),
            content: note_content,
        }))
    }
}

fn list_notes() -> std::io::Result<()> {
    if !Path::new("notes").exists() {
        println!("No notes yet!");
        return Ok(());
    }

    println!("📝 Notes:");
    for entry in fs::read_dir("notes")? {
        let entry = entry?;
        let filename = entry.file_name();
        let title = filename
            .to_str()
            .unwrap()
            .replace(".txt", "")
            .replace('_', " ");

        println!("  - {}", title);
    }

    Ok(())
}

fn main() -> std::io::Result<()> {
    // Create notes
    let note1 = Note::new("Rust Tips", "Always handle errors with Result!\nUse ? operator for cleaner code.");
    note1.save()?;

    let note2 = Note::new("Todo", "1. Learn Rust\n2. Build projects\n3. Contribute to open source");
    note2.save()?;

    let note3 = Note::new("Ideas", "Build a file organizer\nCreate a CLI tool\nWrite a web server");
    note3.save()?;

    // List all notes
    println!();
    list_notes()?;

    // Load and display a note
    println!("\n📄 Loading note: 'Rust Tips'");
    if let Some(note) = Note::load("Rust Tips")? {
        println!("\nTitle: {}", note.title);
        println!("Content:\n{}", note.content);
    }

    Ok(())
}
```

**Output:**
```
✅ Note 'Rust Tips' saved
✅ Note 'Todo' saved
✅ Note 'Ideas' saved

📝 Notes:
  - Rust Tips
  - Todo
  - Ideas

📄 Loading note: 'Rust Tips'

Title: Rust Tips
Content:
Always handle errors with Result!
Use ? operator for cleaner code.
```

## Best Practices

### **1. Always Handle Errors**

```rust
// ❌ Bad: Panics if fails
fs::write("file.txt", "data").unwrap();

// ✅ Good: Handles errors
match fs::write("file.txt", "data") {
    Ok(_) => println!("Success"),
    Err(e) => eprintln!("Error: {}", e),
}
```

### **2. Create Parent Directories**

```rust
use std::fs;
use std::path::Path;

fn write_with_dirs(path: &str, content: &str) -> std::io::Result<()> {
    if let Some(parent) = Path::new(path).parent() {
        fs::create_dir_all(parent)?;
    }

    fs::write(path, content)?;
    Ok(())
}
```

### **3. Use Temporary Files for Safety**

```rust
use std::fs;

fn safe_update(path: &str, content: &str) -> std::io::Result<()> {
    let temp_path = format!("{}.tmp", path);

    // Write to temporary file
    fs::write(&temp_path, content)?;

    // Rename (atomic operation on most systems)
    fs::rename(&temp_path, path)?;

    Ok(())
}
```

### **4. Buffer Large Writes**

```rust
use std::fs::File;
use std::io::{BufWriter, Write};

fn write_many_lines() -> std::io::Result<()> {
    let file = File::create("output.txt")?;
    let mut writer = BufWriter::new(file);

    for i in 1..=10000 {
        writeln!(writer, "Line {}", i)?;
    }

    writer.flush()?;  // Ensure all data written
    Ok(())
}
```

## Key Takeaways

- ✅ `fs::write(path, content)` creates or overwrites file
- ✅ Use `OpenOptions` with `.append(true)` to add content
- ✅ `fs::create_dir_all()` creates nested directories
- ✅ Check `Path::exists()` before overwriting important files
- ✅ Use `.create(true)` to create file if it doesn't exist
- ✅ `writeln!` macro adds newline automatically
- ✅ Use `BufWriter` for better performance with many writes
- ✅ Handle all I/O errors properly
- ✅ Consider using temporary files for important updates
- ✅ Create parent directories before writing files

**Next**: Working with paths and file systems!

---

**Progress**: Module 7, Lesson 3 complete (41/60 lessons total)
