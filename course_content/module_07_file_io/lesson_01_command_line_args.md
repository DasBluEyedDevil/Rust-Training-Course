# Module 7, Lesson 1: Receiving Instructions from Users — Command-Line Arguments

## The Concept: Restaurant Order

Imagine a restaurant:
- **No input**: Kitchen makes default meal
- **With order**: Kitchen makes what you requested: "burger", "fries", "large"

**Command-line arguments** let users specify what they want your program to do.

## What Are Command-Line Arguments?

When you run a program from the terminal, you can pass values:

```bash
cargo run hello world 123
```

- `hello`, `world`, `123` are **arguments**
- Your program can read and use them

## Reading Arguments with std::env

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    println!("Arguments:");
    for (i, arg) in args.iter().enumerate() {
        println!("{}: {}", i, arg);
    }
}
```

**Run it:**
```bash
cargo run hello world
```

**Output:**
```
Arguments:
0: target/debug/your_program
1: hello
2: world
```

**Important**: Argument 0 is always the program name/path!

## Accessing Specific Arguments

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        println!("Please provide a name");
        return;
    }

    let name = &args[1];
    println!("Hello, {}!", name);
}
```

**Run:**
```bash
cargo run Alice
```

**Output:**
```
Hello, Alice!
```

## Handling Multiple Arguments

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 3 {
        println!("Usage: {} <command> <value>", args[0]);
        return;
    }

    let command = &args[1];
    let value = &args[2];

    match command.as_str() {
        "greet" => println!("Hello, {}!", value),
        "repeat" => {
            if let Ok(count) = value.parse::<usize>() {
                for i in 1..=count {
                    println!("{}. Message", i);
                }
            } else {
                println!("Error: '{}' is not a valid number", value);
            }
        }
        _ => println!("Unknown command: {}", command),
    }
}
```

**Run:**
```bash
cargo run greet World
cargo run repeat 3
cargo run unknown test
```

**Output:**
```
Hello, World!
1. Message
2. Message
3. Message
Unknown command: unknown
```

## Better Argument Parsing with clap

For real applications, use the `clap` crate (most popular CLI parser):

### **Step 1: Add clap to Cargo.toml**

```toml
[dependencies]
clap = { version = "4.5", features = ["derive"] }
```

### **Step 2: Use clap**

```rust
use clap::Parser;

/// Simple greeting program
#[derive(Parser)]
#[command(name = "greeter")]
#[command(about = "Greets people", long_about = None)]
struct Args {
    /// Name of the person to greet
    #[arg(short, long)]
    name: String,

    /// Number of times to greet
    #[arg(short, long, default_value_t = 1)]
    count: usize,
}

fn main() {
    let args = Args::parse();

    for _ in 0..args.count {
        println!("Hello, {}!", args.name);
    }
}
```

**Run with different options:**
```bash
cargo run -- --name Alice
cargo run -- --name Bob --count 3
cargo run -- -n Charlie -c 2
cargo run -- --help
```

**Output:**
```
Hello, Alice!

Hello, Bob!
Hello, Bob!
Hello, Bob!

Hello, Charlie!
Hello, Charlie!

Simple greeting program

Usage: greeter --name <NAME> [--count <COUNT>]

Options:
  -n, --name <NAME>     Name of the person to greet
  -c, --count <COUNT>   Number of times to greet [default: 1]
  -h, --help            Print help
```

## Hands-On Practice

```bash
cargo new cli_args
cd cli_args
code .
```

### **Experiment 1: Basic Arguments**

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    println!("Program: {}", args[0]);
    println!("Total arguments: {}", args.len() - 1);

    for (i, arg) in args.iter().skip(1).enumerate() {
        println!("Arg {}: {}", i + 1, arg);
    }
}
```

### **Experiment 2: Calculator**

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() != 4 {
        println!("Usage: {} <num1> <operator> <num2>", args[0]);
        println!("Example: {} 10 + 5", args[0]);
        return;
    }

    let num1: f64 = match args[1].parse() {
        Ok(n) => n,
        Err(_) => {
            println!("Error: '{}' is not a valid number", args[1]);
            return;
        }
    };

    let operator = &args[2];

    let num2: f64 = match args[3].parse() {
        Ok(n) => n,
        Err(_) => {
            println!("Error: '{}' is not a valid number", args[3]);
            return;
        }
    };

    let result = match operator.as_str() {
        "+" => num1 + num2,
        "-" => num1 - num2,
        "*" => num1 * num2,
        "/" => {
            if num2 == 0.0 {
                println!("Error: Division by zero");
                return;
            }
            num1 / num2
        }
        _ => {
            println!("Error: Unknown operator '{}'", operator);
            return;
        }
    };

    println!("{} {} {} = {}", num1, operator, num2, result);
}
```

**Run:**
```bash
cargo run 10 + 5
cargo run 20 - 8
cargo run 6 "*" 7  # Quote * to prevent shell expansion
cargo run 15 / 3
```

### **Experiment 3: File Operation CLI**

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        print_usage(&args[0]);
        return;
    }

    let command = &args[1];

    match command.as_str() {
        "count" => {
            if args.len() != 3 {
                println!("Usage: {} count <text>", args[0]);
                return;
            }
            let text = &args[2];
            println!("Characters: {}", text.len());
            println!("Words: {}", text.split_whitespace().count());
        }
        "reverse" => {
            if args.len() != 3 {
                println!("Usage: {} reverse <text>", args[0]);
                return;
            }
            let text = &args[2];
            let reversed: String = text.chars().rev().collect();
            println!("{}", reversed);
        }
        "upper" => {
            if args.len() != 3 {
                println!("Usage: {} upper <text>", args[0]);
                return;
            }
            let text = &args[2];
            println!("{}", text.to_uppercase());
        }
        _ => {
            println!("Unknown command: {}", command);
            print_usage(&args[0]);
        }
    }
}

fn print_usage(program: &str) {
    println!("Usage: {} <command> [arguments]", program);
    println!("\nCommands:");
    println!("  count <text>    Count characters and words");
    println!("  reverse <text>  Reverse text");
    println!("  upper <text>    Convert to uppercase");
}
```

**Run:**
```bash
cargo run count "hello world"
cargo run reverse "Rust"
cargo run upper "hello"
cargo run help
```

### **Challenge: Task Manager CLI**

Build a simple task manager with commands:

```rust
use std::env;
use std::fs;

const TASKS_FILE: &str = "tasks.txt";

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        print_usage(&args[0]);
        return;
    }

    let command = &args[1];

    match command.as_str() {
        "add" => {
            if args.len() < 3 {
                println!("Usage: {} add <task>", args[0]);
                return;
            }
            let task = args[2..].join(" ");
            add_task(&task);
        }
        "list" => {
            list_tasks();
        }
        "clear" => {
            clear_tasks();
        }
        _ => {
            println!("Unknown command: {}", command);
            print_usage(&args[0]);
        }
    }
}

fn add_task(task: &str) {
    let mut tasks = load_tasks();
    tasks.push(task.to_string());
    save_tasks(&tasks);
    println!("✅ Added: {}", task);
}

fn list_tasks() {
    let tasks = load_tasks();

    if tasks.is_empty() {
        println!("No tasks!");
        return;
    }

    println!("📋 Tasks:");
    for (i, task) in tasks.iter().enumerate() {
        println!("{}. {}", i + 1, task);
    }
}

fn clear_tasks() {
    fs::write(TASKS_FILE, "").expect("Failed to clear tasks");
    println!("✅ All tasks cleared");
}

fn load_tasks() -> Vec<String> {
    match fs::read_to_string(TASKS_FILE) {
        Ok(content) => content
            .lines()
            .filter(|line| !line.is_empty())
            .map(|line| line.to_string())
            .collect(),
        Err(_) => Vec::new(),
    }
}

fn save_tasks(tasks: &[String]) {
    let content = tasks.join("\n");
    fs::write(TASKS_FILE, content).expect("Failed to save tasks");
}

fn print_usage(program: &str) {
    println!("Usage: {} <command> [arguments]", program);
    println!("\nCommands:");
    println!("  add <task>   Add a new task");
    println!("  list         List all tasks");
    println!("  clear        Clear all tasks");
}
```

**Run:**
```bash
cargo run add "Learn Rust"
cargo run add "Build a project"
cargo run list
cargo run add "Master error handling"
cargo run list
cargo run clear
cargo run list
```

**Output:**
```
✅ Added: Learn Rust
✅ Added: Build a project
📋 Tasks:
1. Learn Rust
2. Build a project
✅ Added: Master error handling
📋 Tasks:
1. Learn Rust
2. Build a project
3. Master error handling
✅ All tasks cleared
No tasks!
```

## Common Patterns

### **Pattern 1: Argument Validation**

```rust
fn validate_args(args: &[String]) -> Result<(), String> {
    if args.len() < 2 {
        return Err(String::from("Missing required argument"));
    }

    if args[1].is_empty() {
        return Err(String::from("Argument cannot be empty"));
    }

    Ok(())
}
```

### **Pattern 2: Optional Arguments**

```rust
fn get_arg(args: &[String], index: usize, default: &str) -> String {
    args.get(index)
        .map(|s| s.clone())
        .unwrap_or_else(|| default.to_string())
}

fn main() {
    let args: Vec<String> = env::args().collect();
    let name = get_arg(&args, 1, "World");
    println!("Hello, {}!", name);
}
```

### **Pattern 3: Flag Arguments**

```rust
fn has_flag(args: &[String], flag: &str) -> bool {
    args.iter().any(|arg| arg == flag)
}

fn main() {
    let args: Vec<String> = env::args().collect();

    let verbose = has_flag(&args, "--verbose");

    if verbose {
        println!("Verbose mode enabled");
    }

    println!("Running program...");
}
```

## Key Takeaways

- ✅ `std::env::args()` provides access to command-line arguments
- ✅ First argument (index 0) is always the program name
- ✅ Arguments are strings that may need parsing
- ✅ Always validate argument count and values
- ✅ Use `.parse()` to convert string arguments to numbers
- ✅ Provide helpful error messages and usage instructions
- ✅ Consider using `clap` crate for complex CLI tools
- ✅ Command-line args make programs flexible and scriptable
- ✅ Test with various argument combinations
- ✅ Handle edge cases (missing args, invalid values, etc.)

**Next**: Reading files from disk!

---

**Progress**: Module 7, Lesson 1 complete (39/60 lessons total)
