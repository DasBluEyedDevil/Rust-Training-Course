# Module 9, Lesson 2: Saving Data — File Persistence

## Adding File Storage

Now we'll make tasks persist between program runs by saving them to a JSON file.

## Step 1: Add Storage Module

Create `src/storage.rs`:

```rust
use crate::error::Result;
use crate::manager::TaskManager;
use crate::task::Task;
use serde::{Deserialize, Serialize};
use std::fs;
use std::path::Path;

const DATA_FILE: &str = "tasks.json";

#[derive(Debug, Serialize, Deserialize)]
struct StorageData {
    tasks: Vec<Task>,
    next_id: u32,
}

pub fn save(manager: &TaskManager) -> Result<()> {
    let data = StorageData {
        tasks: manager.tasks.clone(),
        next_id: manager.next_id,
    };

    let json = serde_json::to_string_pretty(&data)?;
    fs::write(DATA_FILE, json)?;

    Ok(())
}

pub fn load() -> Result<TaskManager> {
    if !Path::new(DATA_FILE).exists() {
        return Ok(TaskManager::new());
    }

    let json = fs::read_to_string(DATA_FILE)?;
    let data: StorageData = serde_json::from_str(&json)?;

    Ok(TaskManager {
        tasks: data.tasks,
        next_id: data.next_id,
    })
}

pub fn clear() -> Result<()> {
    if Path::new(DATA_FILE).exists() {
        fs::remove_file(DATA_FILE)?;
    }
    Ok(())
}
```

## Step 2: Update TaskManager

Modify `src/manager.rs` to expose fields needed for storage:

```rust
use crate::error::{Result, TaskError};
use crate::task::{Priority, Status, Task};

pub struct TaskManager {
    pub(crate) tasks: Vec<Task>,
    pub(crate) next_id: u32,
}

// ... rest of implementation stays the same
```

## Step 3: Update Main

Update `src/main.rs` to use persistence:

```rust
mod error;
mod manager;
mod storage;
mod task;

use manager::TaskManager;
use task::Priority;

fn main() {
    println!("=== TaskMaster with Persistence ===\n");

    // Load existing tasks
    let mut manager = match storage::load() {
        Ok(mgr) => {
            println!("✅ Loaded {} tasks from file\n", mgr.count());
            mgr
        }
        Err(e) => {
            eprintln!("⚠️  Failed to load tasks: {}", e);
            eprintln!("Starting with empty task list\n");
            TaskManager::new()
        }
    };

    // Add a new task
    if let Ok(id) = manager.add_task("Complete Rust course".to_string()) {
        manager
            .update_task(id, |task| {
                task.with_priority(Priority::High);
                task.add_tag("learning".to_string());
            })
            .ok();
        println!("✅ Added new task\n");
    }

    // Display all tasks
    println!("Current tasks:");
    for task in manager.list_tasks() {
        println!("  {}", task);
    }

    // Save to file
    match storage::save(&manager) {
        Ok(_) => println!("\n✅ Saved {} tasks to file", manager.count()),
        Err(e) => eprintln!("\n❌ Failed to save tasks: {}", e),
    }
}
```

## Step 4: Test Persistence

```bash
# First run - creates file
cargo run

# Check the file was created
cat tasks.json

# Second run - loads existing data
cargo run

# Third run - should show accumulated tasks
cargo run
```

**tasks.json example:**

```json
{
  "tasks": [
    {
      "id": 1,
      "title": "Complete Rust course",
      "description": null,
      "priority": "High",
      "status": "Todo",
      "tags": ["learning"],
      "created_at": "2024-01-15T10:30:00.000Z",
      "updated_at": "2024-01-15T10:30:00.000Z",
      "due_date": null
    }
  ],
  "next_id": 2
}
```

## Step 5: Add Auto-Save

Update `TaskManager` to auto-save on modifications:

```rust
// In manager.rs

impl TaskManager {
    pub fn add_task(&mut self, title: String) -> Result<u32> {
        if title.trim().is_empty() {
            return Err(TaskError::EmptyTitle);
        }

        let id = self.next_id;
        let task = Task::new(id, title);
        self.tasks.push(task);
        self.next_id += 1;

        // Auto-save
        if let Err(e) = crate::storage::save(self) {
            eprintln!("Warning: Failed to save: {}", e);
        }

        Ok(id)
    }

    pub fn delete_task(&mut self, id: u32) -> Result<()> {
        let pos = self
            .tasks
            .iter()
            .position(|t| t.id == id)
            .ok_or(TaskError::NotFound(id))?;

        self.tasks.remove(pos);

        // Auto-save
        if let Err(e) = crate::storage::save(self) {
            eprintln!("Warning: Failed to save: {}", e);
        }

        Ok(())
    }

    pub fn update_task<F>(&mut self, id: u32, update_fn: F) -> Result<()>
    where
        F: FnOnce(&mut Task),
    {
        let task = self.get_task_mut(id)?;
        update_fn(task);

        // Auto-save
        if let Err(e) = crate::storage::save(self) {
            eprintln!("Warning: Failed to save: {}", e);
        }

        Ok(())
    }
}
```

## Key Concepts

### **Serialization with Serde**

```rust
#[derive(Serialize, Deserialize)]
struct Task {
    // Fields automatically serialized/deserialized
}
```

**Why**: Convert Rust structs to/from JSON automatically.

### **Graceful Degradation**

```rust
let manager = match storage::load() {
    Ok(mgr) => mgr,
    Err(e) => {
        eprintln!("Warning: {}", e);
        TaskManager::new()  // Fallback
    }
};
```

**Why**: Program works even if file doesn't exist.

### **Module Privacy**

```rust
pub struct TaskManager {
    pub(crate) tasks: Vec<Task>,  // Visible within crate only
}
```

**Why**: Expose fields to storage module without making them public.

## Checkpoint

You now have:
- ✅ JSON file persistence
- ✅ Automatic save on modifications
- ✅ Graceful fallback if file missing
- ✅ Pretty-printed JSON format
- ✅ Preserves task IDs across runs

**Next lesson**: Build the command-line interface with clap!

---

**Progress**: Module 9, Lesson 2 complete (50/60 lessons total)
