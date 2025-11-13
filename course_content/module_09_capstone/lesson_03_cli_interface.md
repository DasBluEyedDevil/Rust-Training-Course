# Module 9, Lesson 3: User Interaction — Building the CLI Interface

## Creating a Professional CLI

Now we'll build a complete command-line interface using clap for professional argument parsing.

## Step 1: Define CLI Structure

Create `src/cli.rs`:

```rust
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "taskmaster")]
#[command(about = "A powerful task management CLI", long_about = None)]
#[command(version)]
pub struct Cli {
    #[command(subcommand)]
    pub command: Commands,
}

#[derive(Subcommand)]
pub enum Commands {
    /// Add a new task
    Add {
        /// Task title
        title: String,

        /// Task description
        #[arg(short, long)]
        description: Option<String>,

        /// Priority: low, medium, high, urgent
        #[arg(short, long, default_value = "medium")]
        priority: String,

        /// Tags (comma-separated)
        #[arg(short, long)]
        tags: Option<String>,
    },

    /// List all tasks
    List {
        /// Filter by status: todo, inprogress, done
        #[arg(short, long)]
        status: Option<String>,

        /// Filter by priority
        #[arg(short, long)]
        priority: Option<String>,

        /// Filter by tag
        #[arg(short, long)]
        tag: Option<String>,
    },

    /// Show task details
    Show {
        /// Task ID
        id: u32,
    },

    /// Update a task
    Update {
        /// Task ID
        id: u32,

        /// New title
        #[arg(short = 'T', long)]
        title: Option<String>,

        /// New description
        #[arg(short, long)]
        description: Option<String>,

        /// New priority
        #[arg(short, long)]
        priority: Option<String>,

        /// New status
        #[arg(short, long)]
        status: Option<String>,
    },

    /// Delete a task
    Delete {
        /// Task ID
        id: u32,

        /// Skip confirmation
        #[arg(short, long)]
        force: bool,
    },

    /// Add tag to task
    Tag {
        /// Task ID
        id: u32,

        /// Tag to add
        tag: String,
    },

    /// Remove tag from task
    Untag {
        /// Task ID
        id: u32,

        /// Tag to remove
        tag: String,
    },

    /// Mark task as done
    Done {
        /// Task ID
        id: u32,
    },

    /// Show statistics
    Stats,

    /// Clear all tasks
    Clear {
        /// Skip confirmation
        #[arg(short, long)]
        force: bool,
    },
}
```

## Step 2: Implement CLI Handlers

Create `src/commands.rs`:

```rust
use crate::cli::Commands;
use crate::error::{Result, TaskError};
use crate::manager::TaskManager;
use crate::task::{Priority, Status};
use std::io::{self, Write};

pub fn execute(command: Commands, manager: &mut TaskManager) -> Result<()> {
    match command {
        Commands::Add {
            title,
            description,
            priority,
            tags,
        } => add_task(manager, title, description, priority, tags),

        Commands::List {
            status,
            priority,
            tag,
        } => list_tasks(manager, status, priority, tag),

        Commands::Show { id } => show_task(manager, id),

        Commands::Update {
            id,
            title,
            description,
            priority,
            status,
        } => update_task(manager, id, title, description, priority, status),

        Commands::Delete { id, force } => delete_task(manager, id, force),

        Commands::Tag { id, tag } => add_tag(manager, id, tag),

        Commands::Untag { id, tag } => remove_tag(manager, id, tag),

        Commands::Done { id } => mark_done(manager, id),

        Commands::Stats => show_stats(manager),

        Commands::Clear { force } => clear_all(manager, force),
    }
}

fn add_task(
    manager: &mut TaskManager,
    title: String,
    description: Option<String>,
    priority_str: String,
    tags: Option<String>,
) -> Result<()> {
    let priority = Priority::from_str(&priority_str)
        .ok_or_else(|| TaskError::InvalidPriority(priority_str.clone()))?;

    let id = manager.add_task(title)?;

    manager.update_task(id, |task| {
        task.with_priority(priority);
        if let Some(desc) = description {
            task.with_description(desc);
        }
        if let Some(tags_str) = tags {
            for tag in tags_str.split(',') {
                task.add_tag(tag.trim().to_string());
            }
        }
    })?;

    println!("✅ Created task #{}", id);
    Ok(())
}

fn list_tasks(
    manager: &TaskManager,
    status_filter: Option<String>,
    priority_filter: Option<String>,
    tag_filter: Option<String>,
) -> Result<()> {
    let mut tasks: Vec<_> = manager.list_tasks().iter().collect();

    // Apply filters
    if let Some(status_str) = status_filter {
        let status = Status::from_str(&status_str)
            .ok_or_else(|| TaskError::InvalidStatus(status_str))?;
        tasks.retain(|t| t.status == status);
    }

    if let Some(priority_str) = priority_filter {
        let priority = Priority::from_str(&priority_str)
            .ok_or_else(|| TaskError::InvalidPriority(priority_str))?;
        tasks.retain(|t| t.priority == priority);
    }

    if let Some(tag) = tag_filter {
        tasks.retain(|t| t.tags.iter().any(|t_tag| t_tag.contains(&tag)));
    }

    if tasks.is_empty() {
        println!("No tasks found");
        return Ok(());
    }

    println!("Tasks ({}):", tasks.len());
    for task in tasks {
        println!("  {}", task);
        if !task.tags.is_empty() {
            println!("    Tags: {}", task.tags.join(", "));
        }
    }

    Ok(())
}

fn show_task(manager: &TaskManager, id: u32) -> Result<()> {
    let task = manager.get_task(id)?;

    println!("╔═══════════════════════════════════════════════╗");
    println!("║              Task Details                     ║");
    println!("╠═══════════════════════════════════════════════╣");
    println!("║ ID:          {:30} ║", task.id);
    println!("║ Title:       {:30} ║", task.title);
    println!("║ Status:      {:30} ║", task.status.as_str());
    println!("║ Priority:    {:30} ║", task.priority.as_str());

    if let Some(desc) = &task.description {
        println!("║ Description: {:30} ║", desc);
    }

    if !task.tags.is_empty() {
        println!("║ Tags:        {:30} ║", task.tags.join(", "));
    }

    println!(
        "║ Created:     {:30} ║",
        task.created_at.format("%Y-%m-%d %H:%M").to_string()
    );
    println!(
        "║ Updated:     {:30} ║",
        task.updated_at.format("%Y-%m-%d %H:%M").to_string()
    );
    println!("╚═══════════════════════════════════════════════╝");

    Ok(())
}

fn update_task(
    manager: &mut TaskManager,
    id: u32,
    title: Option<String>,
    description: Option<String>,
    priority_str: Option<String>,
    status_str: Option<String>,
) -> Result<()> {
    let priority = if let Some(p) = priority_str {
        Some(
            Priority::from_str(&p)
                .ok_or_else(|| TaskError::InvalidPriority(p.clone()))?,
        )
    } else {
        None
    };

    let status = if let Some(s) = status_str {
        Some(
            Status::from_str(&s)
                .ok_or_else(|| TaskError::InvalidStatus(s.clone()))?,
        )
    } else {
        None
    };

    manager.update_task(id, |task| {
        if let Some(t) = title {
            task.title = t;
        }
        if let Some(d) = description {
            task.description = Some(d);
        }
        if let Some(p) = priority {
            task.priority = p;
        }
        if let Some(s) = status {
            task.set_status(s);
        }
    })?;

    println!("✅ Updated task #{}", id);
    Ok(())
}

fn delete_task(manager: &mut TaskManager, id: u32, force: bool) -> Result<()> {
    if !force {
        let task = manager.get_task(id)?;
        print!("Delete task '{}' (y/N)? ", task.title);
        io::stdout().flush()?;

        let mut input = String::new();
        io::stdin().read_line(&mut input)?;

        if input.trim().to_lowercase() != "y" {
            println!("Cancelled");
            return Ok(());
        }
    }

    manager.delete_task(id)?;
    println!("✅ Deleted task #{}", id);
    Ok(())
}

fn add_tag(manager: &mut TaskManager, id: u32, tag: String) -> Result<()> {
    manager.update_task(id, |task| {
        task.add_tag(tag.clone());
    })?;

    println!("✅ Added tag '{}' to task #{}", tag, id);
    Ok(())
}

fn remove_tag(manager: &mut TaskManager, id: u32, tag: String) -> Result<()> {
    manager.update_task(id, |task| {
        task.remove_tag(&tag);
    })?;

    println!("✅ Removed tag '{}' from task #{}", tag, id);
    Ok(())
}

fn mark_done(manager: &mut TaskManager, id: u32) -> Result<()> {
    manager.update_task(id, |task| {
        task.set_status(Status::Done);
    })?;

    println!("✅ Marked task #{} as done", id);
    Ok(())
}

fn show_stats(manager: &TaskManager) -> Result<()> {
    let total = manager.count();
    let todo = manager
        .list_tasks()
        .iter()
        .filter(|t| t.status == Status::Todo)
        .count();
    let in_progress = manager
        .list_tasks()
        .iter()
        .filter(|t| t.status == Status::InProgress)
        .count();
    let done = manager
        .list_tasks()
        .iter()
        .filter(|t| t.status == Status::Done)
        .count();

    println!("╔═══════════════════════════════════╗");
    println!("║        Task Statistics            ║");
    println!("╠═══════════════════════════════════╣");
    println!("║ Total:       {:17} ║", total);
    println!("║ Todo:        {:17} ║", todo);
    println!("║ In Progress: {:17} ║", in_progress);
    println!("║ Done:        {:17} ║", done);
    println!("╚═══════════════════════════════════╝");

    if total > 0 {
        let completion = (done as f64 / total as f64) * 100.0;
        println!("\nCompletion: {:.1}%", completion);
    }

    Ok(())
}

fn clear_all(manager: &mut TaskManager, force: bool) -> Result<()> {
    if manager.count() == 0 {
        println!("No tasks to clear");
        return Ok(());
    }

    if !force {
        print!("Clear all {} tasks (y/N)? ", manager.count());
        io::stdout().flush()?;

        let mut input = String::new();
        io::stdin().read_line(&mut input)?;

        if input.trim().to_lowercase() != "y" {
            println!("Cancelled");
            return Ok(());
        }
    }

    let count = manager.count();
    *manager = TaskManager::new();
    crate::storage::save(manager)?;

    println!("✅ Cleared {} tasks", count);
    Ok(())
}
```

## Step 3: Update Main

Update `src/main.rs`:

```rust
mod cli;
mod commands;
mod error;
mod manager;
mod storage;
mod task;

use clap::Parser;
use cli::Cli;

fn main() {
    let cli = Cli::parse();

    let mut manager = match storage::load() {
        Ok(mgr) => mgr,
        Err(e) => {
            eprintln!("Failed to load tasks: {}", e);
            std::process::exit(1);
        }
    };

    if let Err(e) = commands::execute(cli.command, &mut manager) {
        eprintln!("Error: {}", e);
        std::process::exit(1);
    }
}
```

## Step 4: Test the CLI

```bash
# Build the project
cargo build --release

# Add tasks
cargo run -- add "Learn Rust" -p high -t "learning,rust"
cargo run -- add "Build project" -d "Create a CLI app" -p medium

# List tasks
cargo run -- list
cargo run -- list --status todo
cargo run -- list --priority high

# Show details
cargo run -- show 1

# Update task
cargo run -- update 1 --status inprogress
cargo run -- update 2 --priority high

# Tag management
cargo run -- tag 1 important
cargo run -- untag 1 learning

# Mark complete
cargo run -- done 1

# Statistics
cargo run -- stats

# Delete task
cargo run -- delete 2
cargo run -- delete 2 --force  # Skip confirmation

# Clear all
cargo run -- clear
```

## Usage Examples

### **Adding Tasks**

```bash
taskmaster add "Implement feature X"
taskmaster add "Fix bug Y" -p urgent -t "bug,critical"
taskmaster add "Write docs" -d "Document all functions" -p low
```

### **Listing with Filters**

```bash
taskmaster list
taskmaster list --status todo
taskmaster list --priority high
taskmaster list --tag bug
```

### **Task Management**

```bash
taskmaster show 5
taskmaster update 5 --status inprogress --priority high
taskmaster tag 5 review-needed
taskmaster done 5
```

### **Help Command**

```bash
taskmaster --help
taskmaster add --help
taskmaster list --help
```

## Key Concepts

### **Clap Derive API**

```rust
#[derive(Parser)]
#[command(name = "taskmaster")]
pub struct Cli {
    #[command(subcommand)]
    pub command: Commands,
}
```

**Why**: Declarative, type-safe CLI definition with automatic help generation.

### **Error Propagation**

```rust
if let Err(e) = commands::execute(cli.command, &mut manager) {
    eprintln!("Error: {}", e);
    std::process::exit(1);
}
```

**Why**: Clean error handling with informative messages.

### **User Confirmation**

```rust
print!("Delete task (y/N)? ");
io::stdout().flush()?;  // Ensure prompt displays

let mut input = String::new();
io::stdin().read_line(&mut input)?;
```

**Why**: Prevent accidental destructive operations.

## Checkpoint

You now have:
- ✅ Complete CLI with subcommands
- ✅ Argument parsing with clap
- ✅ Filtering and searching
- ✅ Interactive confirmations
- ✅ Professional help messages
- ✅ Error handling throughout

**Next lesson**: Advanced filtering and search capabilities!

---

**Progress**: Module 9, Lesson 3 complete (51/60 lessons total)
