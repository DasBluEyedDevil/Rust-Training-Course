# Module 9 - Capstone Project

# Lesson 1: Project Setup & Core Data Structures

## Project Overview: Task Manager CLI

Welcome to the capstone project! You'll build a complete, professional task management application that brings together everything you've learned:

- ✅ **Structs & Enums** (Module 3)
- ✅ **Ownership & Borrowing** (Module 4)
- ✅ **Collections** (Module 5)
- ✅ **Error Handling** (Module 6)
- ✅ **File I/O** (Module 7)
- ✅ **Traits & Generics** (Module 8)

**What we're building:**
A command-line task manager with:
- Create, read, update, delete tasks
- Priority levels and due dates
- Tags and categories
- Search and filtering
- File persistence
- Statistics and reports

## Step 1: Create the Project

```bash
cargo new taskmaster
cd taskmaster
```

### **Add Dependencies to Cargo.toml**

```toml
[package]
name = "taskmaster"
version = "0.1.0"
edition = "2021"

[dependencies]
clap = { version = "4.5", features = ["derive"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
chrono = "0.4"
```

**Dependencies explained:**
- **clap**: Command-line argument parsing
- **serde**: Serialization/deserialization
- **serde_json**: JSON file format
- **chrono**: Date and time handling

## Step 2: Define Core Data Structures

Create `src/task.rs`:

```rust
use chrono::{DateTime, Local};
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum Priority {
    Low,
    Medium,
    High,
    Urgent,
}

impl Priority {
    pub fn from_str(s: &str) -> Option<Self> {
        match s.to_lowercase().as_str() {
            "low" => Some(Priority::Low),
            "medium" => Some(Priority::Medium),
            "high" => Some(Priority::High),
            "urgent" => Some(Priority::Urgent),
            _ => None,
        }
    }

    pub fn as_str(&self) -> &str {
        match self {
            Priority::Low => "Low",
            Priority::Medium => "Medium",
            Priority::High => "High",
            Priority::Urgent => "Urgent",
        }
    }
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum Status {
    Todo,
    InProgress,
    Done,
}

impl Status {
    pub fn from_str(s: &str) -> Option<Self> {
        match s.to_lowercase().as_str() {
            "todo" => Some(Status::Todo),
            "inprogress" | "in-progress" => Some(Status::InProgress),
            "done" => Some(Status::Done),
            _ => None,
        }
    }

    pub fn as_str(&self) -> &str {
        match self {
            Status::Todo => "Todo",
            Status::InProgress => "In Progress",
            Status::Done => "Done",
        }
    }

    pub fn icon(&self) -> &str {
        match self {
            Status::Todo => "⬜",
            Status::InProgress => "🔄",
            Status::Done => "✅",
        }
    }
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Task {
    pub id: u32,
    pub title: String,
    pub description: Option<String>,
    pub priority: Priority,
    pub status: Status,
    pub tags: Vec<String>,
    pub created_at: DateTime<Local>,
    pub updated_at: DateTime<Local>,
    pub due_date: Option<DateTime<Local>>,
}

impl Task {
    pub fn new(id: u32, title: String) -> Self {
        let now = Local::now();
        Task {
            id,
            title,
            description: None,
            priority: Priority::Medium,
            status: Status::Todo,
            tags: Vec::new(),
            created_at: now,
            updated_at: now,
            due_date: None,
        }
    }

    pub fn with_description(mut self, description: String) -> Self {
        self.description = Some(description);
        self.updated_at = Local::now();
        self
    }

    pub fn with_priority(mut self, priority: Priority) -> Self {
        self.priority = priority;
        self.updated_at = Local::now();
        self
    }

    pub fn with_tags(mut self, tags: Vec<String>) -> Self {
        self.tags = tags;
        self.updated_at = Local::now();
        self
    }

    pub fn set_status(&mut self, status: Status) {
        self.status = status;
        self.updated_at = Local::now();
    }

    pub fn add_tag(&mut self, tag: String) {
        if !self.tags.contains(&tag) {
            self.tags.push(tag);
            self.updated_at = Local::now();
        }
    }

    pub fn remove_tag(&mut self, tag: &str) {
        if let Some(pos) = self.tags.iter().position(|t| t == tag) {
            self.tags.remove(pos);
            self.updated_at = Local::now();
        }
    }
}

impl std::fmt::Display for Task {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(
            f,
            "[{}] #{} - {} | {} | {}",
            self.status.icon(),
            self.id,
            self.title,
            self.priority.as_str(),
            self.status.as_str()
        )
    }
}
```

## Step 3: Create Error Types

Create `src/error.rs`:

```rust
use std::error::Error;
use std::fmt;

#[derive(Debug)]
pub enum TaskError {
    NotFound(u32),
    InvalidId(String),
    InvalidPriority(String),
    InvalidStatus(String),
    IoError(std::io::Error),
    JsonError(serde_json::Error),
    EmptyTitle,
}

impl fmt::Display for TaskError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            TaskError::NotFound(id) => write!(f, "Task #{} not found", id),
            TaskError::InvalidId(s) => write!(f, "Invalid task ID: {}", s),
            TaskError::InvalidPriority(s) => write!(f, "Invalid priority: {}", s),
            TaskError::InvalidStatus(s) => write!(f, "Invalid status: {}", s),
            TaskError::IoError(e) => write!(f, "IO error: {}", e),
            TaskError::JsonError(e) => write!(f, "JSON error: {}", e),
            TaskError::EmptyTitle => write!(f, "Task title cannot be empty"),
        }
    }
}

impl Error for TaskError {}

impl From<std::io::Error> for TaskError {
    fn from(error: std::io::Error) -> Self {
        TaskError::IoError(error)
    }
}

impl From<serde_json::Error> for TaskError {
    fn from(error: serde_json::Error) -> Self {
        TaskError::JsonError(error)
    }
}

pub type Result<T> = std::result::Result<T, TaskError>;
```

## Step 4: Basic Task Manager

Create `src/manager.rs`:

```rust
use crate::error::{Result, TaskError};
use crate::task::{Priority, Status, Task};

pub struct TaskManager {
    tasks: Vec<Task>,
    next_id: u32,
}

impl TaskManager {
    pub fn new() -> Self {
        TaskManager {
            tasks: Vec::new(),
            next_id: 1,
        }
    }

    pub fn add_task(&mut self, title: String) -> Result<u32> {
        if title.trim().is_empty() {
            return Err(TaskError::EmptyTitle);
        }

        let id = self.next_id;
        let task = Task::new(id, title);
        self.tasks.push(task);
        self.next_id += 1;

        Ok(id)
    }

    pub fn get_task(&self, id: u32) -> Result<&Task> {
        self.tasks
            .iter()
            .find(|t| t.id == id)
            .ok_or(TaskError::NotFound(id))
    }

    pub fn get_task_mut(&mut self, id: u32) -> Result<&mut Task> {
        self.tasks
            .iter_mut()
            .find(|t| t.id == id)
            .ok_or(TaskError::NotFound(id))
    }

    pub fn list_tasks(&self) -> &[Task] {
        &self.tasks
    }

    pub fn update_task<F>(&mut self, id: u32, update_fn: F) -> Result<()>
    where
        F: FnOnce(&mut Task),
    {
        let task = self.get_task_mut(id)?;
        update_fn(task);
        Ok(())
    }

    pub fn delete_task(&mut self, id: u32) -> Result<()> {
        let pos = self
            .tasks
            .iter()
            .position(|t| t.id == id)
            .ok_or(TaskError::NotFound(id))?;

        self.tasks.remove(pos);
        Ok(())
    }

    pub fn count(&self) -> usize {
        self.tasks.len()
    }
}

impl Default for TaskManager {
    fn default() -> Self {
        Self::new()
    }
}
```

## Step 5: Main Program (Basic Version)

Update `src/main.rs`:

```rust
mod error;
mod manager;
mod task;

use manager::TaskManager;
use task::{Priority, Status};

fn main() {
    println!("=== TaskMaster Demo ===\n");

    let mut manager = TaskManager::new();

    // Add tasks
    let id1 = manager.add_task("Learn Rust".to_string()).unwrap();
    let id2 = manager.add_task("Build a project".to_string()).unwrap();
    let id3 = manager.add_task("Master error handling".to_string()).unwrap();

    println!("✅ Created {} tasks\n", manager.count());

    // Update tasks
    manager
        .update_task(id1, |task| {
            task.with_priority(Priority::High);
            task.set_status(Status::InProgress);
            task.add_tag("learning".to_string());
        })
        .unwrap();

    manager
        .update_task(id2, |task| {
            task.with_priority(Priority::Medium);
            task.add_tag("project".to_string());
        })
        .unwrap();

    // List all tasks
    println!("All tasks:");
    for task in manager.list_tasks() {
        println!("  {}", task);
    }

    // Get specific task
    println!("\n=== Task Details ===");
    if let Ok(task) = manager.get_task(id1) {
        println!("Task #{}:", task.id);
        println!("  Title: {}", task.title);
        println!("  Priority: {}", task.priority.as_str());
        println!("  Status: {}", task.status.as_str());
        println!("  Tags: {:?}", task.tags);
        println!("  Created: {}", task.created_at.format("%Y-%m-%d %H:%M"));
    }

    // Delete task
    manager.delete_task(id3).unwrap();
    println!("\n✅ Deleted task #{}", id3);
    println!("Remaining tasks: {}", manager.count());
}
```

## Step 6: Build and Test

```bash
cargo build
cargo run
```

**Expected Output:**

```
=== TaskMaster Demo ===

✅ Created 3 tasks

All tasks:
  [🔄] #1 - Learn Rust | High | In Progress
  [⬜] #2 - Build a project | Medium | Todo
  [⬜] #3 - Master error handling | Medium | Todo

=== Task Details ===
Task #1:
  Title: Learn Rust
  Priority: High
  Status: In Progress
  Tags: ["learning"]
  Created: 2024-01-15 10:30

✅ Deleted task #3
Remaining tasks: 2
```

## Understanding the Code

### **Builder Pattern**

```rust
let task = Task::new(id, title)
    .with_description("Details".to_string())
    .with_priority(Priority::High);
```

**Why**: Chainable, readable task construction.

### **Generic Update Function**

```rust
fn update_task<F>(&mut self, id: u32, update_fn: F) -> Result<()>
where
    F: FnOnce(&mut Task),
```

**Why**: Flexible task updates without exposing mutable references.

### **Custom Error Types**

```rust
enum TaskError {
    NotFound(u32),
    InvalidId(String),
    // ...
}
```

**Why**: Specific, actionable error information with context.

### **Trait Implementations**

```rust
impl Display for Task { }
impl From<std::io::Error> for TaskError { }
```

**Why**: Integrates with Rust ecosystem (println!, ? operator).

## Checkpoint

You now have:
- ✅ Core data structures (Task, Priority, Status)
- ✅ Custom error types with From trait
- ✅ Task manager with CRUD operations
- ✅ Builder pattern for task creation
- ✅ Display trait for pretty printing

**Next lesson**: Add file persistence to save and load tasks!

---

**Progress**: Module 9, Lesson 1 complete (49/60 lessons total)
