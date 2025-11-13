# Module 9, Lesson 4: Power Features — Advanced Filtering & Search

## Adding Search and Advanced Filtering

Let's add powerful search and filtering capabilities using iterators and traits.

## Step 1: Add Filterable Trait

Update `src/task.rs` to add searching:

```rust
impl Task {
    pub fn matches_search(&self, query: &str) -> bool {
        let query_lower = query.to_lowercase();

        self.title.to_lowercase().contains(&query_lower)
            || self
                .description
                .as_ref()
                .map(|d| d.to_lowercase().contains(&query_lower))
                .unwrap_or(false)
            || self.tags.iter().any(|t| t.to_lowercase().contains(&query_lower))
    }

    pub fn is_overdue(&self) -> bool {
        if let Some(due) = self.due_date {
            due < chrono::Local::now()
        } else {
            false
        }
    }

    pub fn days_until_due(&self) -> Option<i64> {
        self.due_date.map(|due| {
            let duration = due.signed_duration_since(chrono::Local::now());
            duration.num_days()
        })
    }
}
```

## Step 2: Add Filter Builder

Create `src/filter.rs`:

```rust
use crate::task::{Priority, Status, Task};

pub struct TaskFilter<'a> {
    tasks: &'a [Task],
}

impl<'a> TaskFilter<'a> {
    pub fn new(tasks: &'a [Task]) -> Self {
        TaskFilter { tasks }
    }

    pub fn by_status(self, status: Status) -> Vec<&'a Task> {
        self.tasks
            .iter()
            .filter(|t| t.status == status)
            .collect()
    }

    pub fn by_priority(self, priority: Priority) -> Vec<&'a Task> {
        self.tasks
            .iter()
            .filter(|t| t.priority == priority)
            .collect()
    }

    pub fn by_tag(self, tag: &str) -> Vec<&'a Task> {
        self.tasks
            .iter()
            .filter(|t| t.tags.iter().any(|t_tag| t_tag.contains(tag)))
            .collect()
    }

    pub fn search(self, query: &str) -> Vec<&'a Task> {
        self.tasks
            .iter()
            .filter(|t| t.matches_search(query))
            .collect()
    }

    pub fn overdue(self) -> Vec<&'a Task> {
        self.tasks.iter().filter(|t| t.is_overdue()).collect()
    }

    pub fn due_soon(self, days: i64) -> Vec<&'a Task> {
        self.tasks
            .iter()
            .filter(|t| {
                if let Some(days_until) = t.days_until_due() {
                    days_until >= 0 && days_until <= days
                } else {
                    false
                }
            })
            .collect()
    }

    pub fn sort_by_priority(mut self) -> Vec<&'a Task> {
        let mut tasks: Vec<&Task> = self.tasks.iter().collect();
        tasks.sort_by(|a, b| {
            let priority_order = |p: &Priority| match p {
                Priority::Urgent => 0,
                Priority::High => 1,
                Priority::Medium => 2,
                Priority::Low => 3,
            };
            priority_order(&a.priority).cmp(&priority_order(&b.priority))
        });
        tasks
    }

    pub fn sort_by_due_date(mut self) -> Vec<&'a Task> {
        let mut tasks: Vec<&Task> = self.tasks.iter().collect();
        tasks.sort_by(|a, b| {
            match (a.due_date, b.due_date) {
                (Some(a_due), Some(b_due)) => a_due.cmp(&b_due),
                (Some(_), None) => std::cmp::Ordering::Less,
                (None, Some(_)) => std::cmp::Ordering::Greater,
                (None, None) => std::cmp::Ordering::Equal,
            }
        });
        tasks
    }
}
```

## Step 3: Add Search Command

Update `src/cli.rs`:

```rust
#[derive(Subcommand)]
pub enum Commands {
    // ... existing commands ...

    /// Search tasks
    Search {
        /// Search query
        query: String,
    },

    /// Show overdue tasks
    Overdue,

    /// Show tasks due soon
    DueSoon {
        /// Number of days
        #[arg(default_value = "7")]
        days: i64,
    },
}
```

## Step 4: Implement Search Commands

Update `src/commands.rs`:

```rust
use crate::filter::TaskFilter;

pub fn execute(command: Commands, manager: &mut TaskManager) -> Result<()> {
    match command {
        // ... existing commands ...

        Commands::Search { query } => search_tasks(manager, query),
        Commands::Overdue => show_overdue(manager),
        Commands::DueSoon { days } => show_due_soon(manager, days),
    }
}

fn search_tasks(manager: &TaskManager, query: String) -> Result<()> {
    let filter = TaskFilter::new(manager.list_tasks());
    let results = filter.search(&query);

    if results.is_empty() {
        println!("No tasks found matching '{}'", query);
        return Ok(());
    }

    println!("Found {} task(s) matching '{}':", results.len(), query);
    for task in results {
        println!("  {}", task);
    }

    Ok(())
}

fn show_overdue(manager: &TaskManager) -> Result<()> {
    let filter = TaskFilter::new(manager.list_tasks());
    let overdue = filter.overdue();

    if overdue.is_empty() {
        println!("✅ No overdue tasks");
        return Ok(());
    }

    println!("⚠️  {} overdue task(s):", overdue.len());
    for task in overdue {
        if let Some(days) = task.days_until_due() {
            println!("  {} (overdue by {} days)", task, days.abs());
        } else {
            println!("  {}", task);
        }
    }

    Ok(())
}

fn show_due_soon(manager: &TaskManager, days: i64) -> Result<()> {
    let filter = TaskFilter::new(manager.list_tasks());
    let due_soon = filter.due_soon(days);

    if due_soon.is_empty() {
        println!("No tasks due in the next {} days", days);
        return Ok(());
    }

    println!("{} task(s) due in the next {} days:", due_soon.len(), days);
    for task in due_soon {
        if let Some(days_until) = task.days_until_due() {
            println!("  {} (due in {} days)", task, days_until);
        }
    }

    Ok(())
}
```

## Step 5: Add Due Date Support

Update `src/cli.rs` to add due date to Add command:

```rust
Commands::Add {
    // ... existing fields ...

    /// Due date (YYYY-MM-DD)
    #[arg(short = 'D', long)]
    due: Option<String>,
},
```

Update `src/commands.rs` add_task function:

```rust
fn add_task(
    manager: &mut TaskManager,
    title: String,
    description: Option<String>,
    priority_str: String,
    tags: Option<String>,
    due_str: Option<String>,
) -> Result<()> {
    let priority = Priority::from_str(&priority_str)
        .ok_or_else(|| TaskError::InvalidPriority(priority_str.clone()))?;

    let due_date = if let Some(date_str) = due_str {
        Some(
            chrono::NaiveDate::parse_from_str(&date_str, "%Y-%m-%d")
                .map_err(|_| TaskError::InvalidDate(date_str.clone()))?
                .and_hms_opt(23, 59, 59)
                .ok_or_else(|| TaskError::InvalidDate(date_str.clone()))?
                .and_local_timezone(chrono::Local)
                .single()
                .ok_or_else(|| TaskError::InvalidDate(date_str))?,
        )
    } else {
        None
    };

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
        if let Some(due) = due_date {
            task.due_date = Some(due);
        }
    })?;

    println!("✅ Created task #{}", id);
    Ok(())
}
```

Add InvalidDate to `src/error.rs`:

```rust
#[derive(Debug)]
pub enum TaskError {
    // ... existing variants ...
    InvalidDate(String),
}

impl fmt::Display for TaskError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            // ... existing cases ...
            TaskError::InvalidDate(s) => write!(f, "Invalid date format: {} (use YYYY-MM-DD)", s),
        }
    }
}
```

## Usage Examples

### **Advanced Search**

```bash
# Search across title, description, and tags
taskmaster search rust
taskmaster search "bug fix"
taskmaster search urgent
```

### **Due Date Management**

```bash
# Add task with due date
taskmaster add "Submit report" -D 2024-12-31 -p high

# Show overdue tasks
taskmaster overdue

# Show tasks due in next 7 days
taskmaster due-soon

# Show tasks due in next 30 days
taskmaster due-soon 30
```

### **Combined Filtering**

```bash
# High priority todos
taskmaster list --status todo --priority high

# Tasks with specific tag
taskmaster list --tag urgent

# Search within filtered results
taskmaster search "bug" | grep -i "high"
```

## Key Concepts

### **Fluent Filter API**

```rust
TaskFilter::new(tasks)
    .by_status(Status::Todo)
    .sort_by_priority()
```

**Why**: Composable, readable filtering with method chaining.

### **Iterator Chains**

```rust
tasks
    .iter()
    .filter(|t| t.matches_search(query))
    .collect()
```

**Why**: Efficient, lazy evaluation with functional style.

### **Option Handling**

```rust
task.days_until_due().map(|days| {
    // Process days value
})
```

**Why**: Safe handling of optional due dates.

## Checkpoint

You now have:
- ✅ Full-text search across all task fields
- ✅ Advanced filtering by multiple criteria
- ✅ Due date support with parsing
- ✅ Overdue task detection
- ✅ Sortable task lists
- ✅ Composable filter API

**Next lesson**: Statistics, reports, and data visualization!

---

**Progress**: Module 9, Lesson 4 complete (52/60 lessons total)
