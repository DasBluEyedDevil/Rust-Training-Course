# Module 9, Lesson 5: Insights — Statistics and Reports

## Generating Insights from Task Data

Let's add comprehensive statistics and reporting capabilities.

## Step 1: Create Stats Module

Create `src/stats.rs`:

```rust
use crate::task::{Priority, Status, Task};
use std::collections::HashMap;

pub struct TaskStats {
    pub total: usize,
    pub by_status: HashMap<Status, usize>,
    pub by_priority: HashMap<Priority, usize>,
    pub completed_percentage: f64,
    pub overdue_count: usize,
    pub average_age_days: f64,
}

impl TaskStats {
    pub fn from_tasks(tasks: &[Task]) -> Self {
        let total = tasks.len();

        let mut by_status = HashMap::new();
        for status in [Status::Todo, Status::InProgress, Status::Done] {
            let count = tasks.iter().filter(|t| t.status == status).count();
            by_status.insert(status, count);
        }

        let mut by_priority = HashMap::new();
        for priority in [
            Priority::Low,
            Priority::Medium,
            Priority::High,
            Priority::Urgent,
        ] {
            let count = tasks.iter().filter(|t| t.priority == priority).count();
            by_priority.insert(priority, count);
        }

        let done_count = *by_status.get(&Status::Done).unwrap_or(&0);
        let completed_percentage = if total > 0 {
            (done_count as f64 / total as f64) * 100.0
        } else {
            0.0
        };

        let overdue_count = tasks.iter().filter(|t| t.is_overdue()).count();

        let total_age_days: i64 = tasks
            .iter()
            .map(|t| {
                let duration = chrono::Local::now().signed_duration_since(t.created_at);
                duration.num_days()
            })
            .sum();

        let average_age_days = if total > 0 {
            total_age_days as f64 / total as f64
        } else {
            0.0
        };

        TaskStats {
            total,
            by_status,
            by_priority,
            completed_percentage,
            overdue_count,
            average_age_days,
        }
    }

    pub fn display(&self) {
        println!("╔═══════════════════════════════════════════╗");
        println!("║         Task Statistics Report            ║");
        println!("╠═══════════════════════════════════════════╣");
        println!("║                                           ║");
        println!("║ Overview                                  ║");
        println!("║   Total tasks:        {:17} ║", self.total);
        println!(
            "║   Completion rate:    {:15.1}% ║",
            self.completed_percentage
        );
        println!("║   Overdue tasks:      {:17} ║", self.overdue_count);
        println!(
            "║   Average age:        {:13.1} days ║",
            self.average_age_days
        );
        println!("║                                           ║");
        println!("║ By Status                                 ║");
        println!(
            "║   Todo:               {:17} ║",
            self.by_status.get(&Status::Todo).unwrap_or(&0)
        );
        println!(
            "║   In Progress:        {:17} ║",
            self.by_status.get(&Status::InProgress).unwrap_or(&0)
        );
        println!(
            "║   Done:               {:17} ║",
            self.by_status.get(&Status::Done).unwrap_or(&0)
        );
        println!("║                                           ║");
        println!("║ By Priority                               ║");
        println!(
            "║   Urgent:             {:17} ║",
            self.by_priority.get(&Priority::Urgent).unwrap_or(&0)
        );
        println!(
            "║   High:               {:17} ║",
            self.by_priority.get(&Priority::High).unwrap_or(&0)
        );
        println!(
            "║   Medium:             {:17} ║",
            self.by_priority.get(&Priority::Medium).unwrap_or(&0)
        );
        println!(
            "║   Low:                {:17} ║",
            self.by_priority.get(&Priority::Low).unwrap_or(&0)
        );
        println!("╚═══════════════════════════════════════════╝");
    }

    pub fn progress_bar(&self, width: usize) -> String {
        let filled = ((self.completed_percentage / 100.0) * width as f64) as usize;
        let empty = width - filled;

        format!(
            "[{}{}] {:.1}%",
            "█".repeat(filled),
            "░".repeat(empty),
            self.completed_percentage
        )
    }
}

pub fn tag_distribution(tasks: &[Task]) -> HashMap<String, usize> {
    let mut distribution = HashMap::new();

    for task in tasks {
        for tag in &task.tags {
            *distribution.entry(tag.clone()).or_insert(0) += 1;
        }
    }

    distribution
}

pub fn productivity_summary(tasks: &[Task], days: i64) -> ProductivitySummary {
    let cutoff = chrono::Local::now() - chrono::Duration::days(days);

    let recent_completed = tasks
        .iter()
        .filter(|t| t.status == Status::Done && t.updated_at >= cutoff)
        .count();

    let recent_created = tasks
        .iter()
        .filter(|t| t.created_at >= cutoff)
        .count();

    let tasks_per_day = recent_completed as f64 / days as f64;

    ProductivitySummary {
        days,
        tasks_completed: recent_completed,
        tasks_created: recent_created,
        completion_rate: tasks_per_day,
    }
}

pub struct ProductivitySummary {
    pub days: i64,
    pub tasks_completed: usize,
    pub tasks_created: usize,
    pub completion_rate: f64,
}

impl ProductivitySummary {
    pub fn display(&self) {
        println!("╔═══════════════════════════════════════════╗");
        println!("║      Productivity Report (Last {} days)  ║", self.days);
        println!("╠═══════════════════════════════════════════╣");
        println!(
            "║ Tasks completed:      {:17} ║",
            self.tasks_completed
        );
        println!("║ Tasks created:        {:17} ║", self.tasks_created);
        println!(
            "║ Completion rate:      {:13.2}/day ║",
            self.completion_rate
        );
        println!("╚═══════════════════════════════════════════╝");
    }
}
```

## Step 2: Update Commands

Update `src/cli.rs`:

```rust
Commands::Stats {
    /// Show detailed statistics
    #[arg(short, long)]
    detailed: bool,
},

/// Show productivity report
Productivity {
    /// Number of days to analyze
    #[arg(default_value = "7")]
    days: i64,
},

/// Show tag distribution
Tags,

/// Export tasks to file
Export {
    /// Output file
    #[arg(default_value = "tasks_export.json")]
    file: String,
},
```

Update `src/commands.rs`:

```rust
use crate::stats::{productivity_summary, tag_distribution, TaskStats};

Commands::Stats { detailed } => {
    show_stats(manager, detailed)
}

Commands::Productivity { days } => {
    show_productivity(manager, days)
}

Commands::Tags => show_tags(manager),

Commands::Export { file } => export_tasks(manager, &file),

fn show_stats(manager: &TaskManager, detailed: bool) -> Result<()> {
    let stats = TaskStats::from_tasks(manager.list_tasks());

    if detailed {
        stats.display();
    } else {
        println!("Tasks: {} total", stats.total);
        println!(
            "Progress: {}",
            stats.progress_bar(30)
        );

        if stats.overdue_count > 0 {
            println!("⚠️  {} overdue", stats.overdue_count);
        }
    }

    Ok(())
}

fn show_productivity(manager: &TaskManager, days: i64) -> Result<()> {
    let summary = productivity_summary(manager.list_tasks(), days);
    summary.display();
    Ok(())
}

fn show_tags(manager: &TaskManager) -> Result<()> {
    let distribution = tag_distribution(manager.list_tasks());

    if distribution.is_empty() {
        println!("No tags found");
        return Ok(());
    }

    let mut tags: Vec<_> = distribution.into_iter().collect();
    tags.sort_by(|a, b| b.1.cmp(&a.1)); // Sort by count descending

    println!("Tag Distribution:");
    for (tag, count) in tags {
        println!("  {:20} {}", tag, count);
    }

    Ok(())
}

fn export_tasks(manager: &TaskManager, filename: &str) -> Result<()> {
    let json = serde_json::to_string_pretty(manager.list_tasks())?;
    std::fs::write(filename, json)?;

    println!("✅ Exported {} tasks to {}", manager.count(), filename);
    Ok(())
}
```

## Step 3: Add Report Command

Create a comprehensive report combining all stats:

```rust
Commands::Report => generate_report(manager),

fn generate_report(manager: &TaskManager) -> Result<()> {
    let stats = TaskStats::from_tasks(manager.list_tasks());

    println!("\n═══════════════════════════════════════════════");
    println!("            TASKMASTER REPORT");
    println!("═══════════════════════════════════════════════\n");

    // Overview
    stats.display();

    // Productivity
    println!();
    let productivity = productivity_summary(manager.list_tasks(), 7);
    productivity.display();

    // Tag distribution
    println!();
    let distribution = tag_distribution(manager.list_tasks());
    if !distribution.is_empty() {
        println!("╔═══════════════════════════════════════════╗");
        println!("║           Tag Distribution                ║");
        println!("╠═══════════════════════════════════════════╣");

        let mut tags: Vec<_> = distribution.into_iter().collect();
        tags.sort_by(|a, b| b.1.cmp(&a.1));
        tags.truncate(5); // Top 5

        for (tag, count) in tags {
            println!("║ {:30} {:8} ║", tag, count);
        }
        println!("╚═══════════════════════════════════════════╝");
    }

    // Urgent items
    println!();
    let urgent = manager
        .list_tasks()
        .iter()
        .filter(|t| t.priority == Priority::Urgent && t.status != Status::Done)
        .collect::<Vec<_>>();

    if !urgent.is_empty() {
        println!("╔═══════════════════════════════════════════╗");
        println!("║           Urgent Items                    ║");
        println!("╠═══════════════════════════════════════════╣");
        for task in urgent {
            println!("║ {:<42} ║", format!("#{} - {}", task.id, task.title));
        }
        println!("╚═══════════════════════════════════════════╝");
    }

    Ok(())
}
```

## Usage Examples

### **Quick Stats**

```bash
taskmaster stats
# Output:
# Tasks: 25 total
# Progress: [████████████░░░░░░░░░░░░░░░░░░] 40.0%
```

### **Detailed Statistics**

```bash
taskmaster stats --detailed
```

### **Productivity Analysis**

```bash
taskmaster productivity       # Last 7 days
taskmaster productivity 30    # Last 30 days
```

### **Tag Analysis**

```bash
taskmaster tags
# Output:
# Tag Distribution:
#   bug                  5
#   feature              3
#   documentation        2
```

### **Complete Report**

```bash
taskmaster report
```

### **Export Data**

```bash
taskmaster export
taskmaster export tasks_backup.json
```

## Key Concepts

### **Aggregate Statistics**

```rust
let total_age: i64 = tasks
    .iter()
    .map(|t| calculate_age(t))
    .sum();
```

**Why**: Iterator methods for efficient data aggregation.

### **HashMap for Grouping**

```rust
let mut by_status = HashMap::new();
for task in tasks {
    *by_status.entry(task.status).or_insert(0) += 1;
}
```

**Why**: Count occurrences by category efficiently.

### **Visual Progress Indicators**

```rust
fn progress_bar(&self, width: usize) -> String {
    let filled = ((self.percentage / 100.0) * width as f64) as usize;
    format!("[{}{}]", "█".repeat(filled), "░".repeat(width - filled))
}
```

**Why**: User-friendly visual feedback in CLI.

## Checkpoint

You now have:
- ✅ Comprehensive statistics reports
- ✅ Progress visualization
- ✅ Productivity tracking
- ✅ Tag distribution analysis
- ✅ Data export functionality
- ✅ Multi-faceted reporting

**Next lesson**: Final polish and deployment!

---

**Progress**: Module 9, Lesson 5 complete (53/60 lessons total)
