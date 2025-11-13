# Module 9, Lesson 6: Finishing Touches — Testing & Deployment

## Final Polish and Production Readiness

Let's add the finishing touches to make TaskMaster production-ready.

## Step 1: Add Unit Tests

Create `src/task_tests.rs`:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use crate::task::{Priority, Status, Task};

    #[test]
    fn test_task_creation() {
        let task = Task::new(1, "Test task".to_string());
        assert_eq!(task.id, 1);
        assert_eq!(task.title, "Test task");
        assert_eq!(task.status, Status::Todo);
        assert_eq!(task.priority, Priority::Medium);
    }

    #[test]
    fn test_task_builder() {
        let task = Task::new(1, "Test".to_string())
            .with_priority(Priority::High)
            .with_description("Description".to_string());

        assert_eq!(task.priority, Priority::High);
        assert_eq!(task.description, Some("Description".to_string()));
    }

    #[test]
    fn test_task_search() {
        let task = Task::new(1, "Learn Rust".to_string())
            .with_description("Study ownership".to_string());

        assert!(task.matches_search("rust"));
        assert!(task.matches_search("ownership"));
        assert!(!task.matches_search("python"));
    }

    #[test]
    fn test_tag_management() {
        let mut task = Task::new(1, "Test".to_string());

        task.add_tag("important".to_string());
        assert_eq!(task.tags.len(), 1);

        task.add_tag("urgent".to_string());
        assert_eq!(task.tags.len(), 2);

        task.remove_tag("important");
        assert_eq!(task.tags.len(), 1);
        assert!(task.tags.contains(&"urgent".to_string()));
    }
}
```

Run tests:

```bash
cargo test
```

## Step 2: Add Configuration

Create `src/config.rs`:

```rust
use serde::{Deserialize, Serialize};
use std::fs;
use std::path::PathBuf;

#[derive(Debug, Serialize, Deserialize)]
pub struct Config {
    pub data_file: String,
    pub auto_save: bool,
    pub default_priority: String,
}

impl Default for Config {
    fn default() -> Self {
        Config {
            data_file: "tasks.json".to_string(),
            auto_save: true,
            default_priority: "medium".to_string(),
        }
    }
}

impl Config {
    pub fn load() -> Self {
        let config_path = Self::config_path();

        if config_path.exists() {
            if let Ok(content) = fs::read_to_string(&config_path) {
                if let Ok(config) = serde_json::from_str(&content) {
                    return config;
                }
            }
        }

        Config::default()
    }

    pub fn save(&self) -> std::io::Result<()> {
        let config_path = Self::config_path();

        if let Some(parent) = config_path.parent() {
            fs::create_dir_all(parent)?;
        }

        let json = serde_json::to_string_pretty(self)?;
        fs::write(&config_path, json)?;

        Ok(())
    }

    fn config_path() -> PathBuf {
        dirs::config_dir()
            .unwrap_or_else(|| PathBuf::from("."))
            .join("taskmaster")
            .join("config.json")
    }
}
```

Add `dirs` dependency to `Cargo.toml`:

```toml
[dependencies]
dirs = "5.0"
```

## Step 3: Add README

Create a comprehensive `README.md`:

```markdown
# TaskMaster

A powerful command-line task manager built with Rust.

## Features

- ✅ Create, update, and delete tasks
- ✅ Priority levels (Low, Medium, High, Urgent)
- ✅ Status tracking (Todo, In Progress, Done)
- ✅ Tags for organization
- ✅ Due dates with overdue detection
- ✅ Full-text search
- ✅ Statistics and productivity reports
- ✅ JSON file persistence
- ✅ Export/import functionality

## Installation

### From Source

```bash
git clone https://github.com/yourusername/taskmaster.git
cd taskmaster
cargo build --release
cargo install --path .
```

## Usage

### Basic Commands

```bash
# Add a task
taskmaster add "Learn Rust" -p high -t learning

# List tasks
taskmaster list
taskmaster list --status todo
taskmaster list --priority high

# Show task details
taskmaster show 1

# Update task
taskmaster update 1 --status inprogress
taskmaster update 1 --priority urgent

# Mark as done
taskmaster done 1

# Delete task
taskmaster delete 1
```

### Advanced Features

```bash
# Search tasks
taskmaster search "rust"

# Show overdue tasks
taskmaster overdue

# Show tasks due soon
taskmaster due-soon 7

# Statistics
taskmaster stats
taskmaster stats --detailed

# Productivity report
taskmaster productivity 30

# Complete report
taskmaster report

# Export data
taskmaster export tasks_backup.json
```

## Configuration

TaskMaster stores configuration in `~/.config/taskmaster/config.json`:

```json
{
  "data_file": "tasks.json",
  "auto_save": true,
  "default_priority": "medium"
}
```

## License

MIT

## Contributing

Contributions welcome! Please open an issue or PR.
```

## Step 4: Build Release Binary

```bash
# Build optimized release binary
cargo build --release

# Binary located at: target/release/taskmaster

# Install globally (Unix-like systems)
cargo install --path .

# Create distributable package
cargo build --release
strip target/release/taskmaster  # Reduce binary size
```

## Step 5: Add Shell Completions

Add to `src/main.rs`:

```rust
use clap::CommandFactory;
use clap_complete::{generate_to, shells::*};

// Generate shell completions
fn generate_completions() {
    let mut cmd = Cli::command();
    let bin_name = "taskmaster";

    let outdir = std::env::var_os("OUT_DIR").unwrap();

    generate_to(Bash, &mut cmd, bin_name, &outdir).ok();
    generate_to(Zsh, &mut cmd, bin_name, &outdir).ok();
    generate_to(Fish, &mut cmd, bin_name, &outdir).ok();
}
```

Add dependency:

```toml
[dependencies]
clap_complete = "4.5"
```

## Usage as Library

TaskMaster can also be used as a library:

```rust
use taskmaster::{TaskManager, Task, Priority};

fn main() {
    let mut manager = TaskManager::new();

    let id = manager.add_task("My task".to_string()).unwrap();

    manager.update_task(id, |task| {
        task.with_priority(Priority::High);
    }).unwrap();

    for task in manager.list_tasks() {
        println!("{}", task);
    }
}
```

## Complete Example: Daily Workflow

```bash
#!/bin/bash
# daily_tasks.sh

# Morning routine
taskmaster list --status todo --priority urgent
taskmaster overdue
taskmaster due-soon 1

# During work
taskmaster add "Review PR #123" -p high -t review
taskmaster update 15 --status inprogress
taskmaster done 12

# End of day
taskmaster stats --detailed
taskmaster productivity 1
taskmaster export daily_backup.json
```

## Troubleshooting

### Tasks not persisting

Check file permissions for `tasks.json`:

```bash
ls -l tasks.json
chmod 644 tasks.json
```

### Performance with many tasks

TaskMaster efficiently handles thousands of tasks. If experiencing slowness:

```bash
# Export and archive old completed tasks
taskmaster export archive.json
taskmaster list --status done | # filter and delete old ones
```

## Capstone Project Complete! 🎉

You've built a professional-grade CLI application demonstrating:

- ✅ **Structs & Enums**: Task, Priority, Status
- ✅ **Error Handling**: Custom errors with From trait
- ✅ **File I/O**: JSON persistence
- ✅ **Traits**: Display, Serialize, Deserialize
- ✅ **Generics**: Flexible functions
- ✅ **Iterators**: Filtering, mapping, collecting
- ✅ **CLI Parsing**: clap with subcommands
- ✅ **Testing**: Unit tests
- ✅ **Documentation**: Comprehensive README

## Next Steps

**Optional enhancements:**

1. **Database backend**: SQLite instead of JSON
2. **Sync**: Cloud synchronization
3. **Web interface**: REST API with web UI
4. **Mobile app**: Share data with mobile client
5. **AI features**: Smart task suggestions
6. **Team features**: Shared task lists
7. **Notifications**: Desktop/email reminders
8. **Time tracking**: Log time spent on tasks
9. **Recurring tasks**: Scheduled repetition
10. **Subtasks**: Task hierarchies

## Deployment Options

### **As CLI Tool**

```bash
cargo install --path .
```

### **Distribute Binary**

```bash
# Create releases for multiple platforms
cargo build --release --target x86_64-unknown-linux-gnu
cargo build --release --target x86_64-apple-darwin
cargo build --release --target x86_64-pc-windows-msvc
```

### **Publish to crates.io**

```bash
cargo publish
```

---

**Progress**: Module 9, Lesson 6 complete (54/60 lessons total)

**Next**: Quiz to test your understanding of building complete applications!
