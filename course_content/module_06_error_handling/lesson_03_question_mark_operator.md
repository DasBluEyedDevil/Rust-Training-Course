# Module 6, Lesson 3: Clean Error Handling — The `?` Operator Mastery

## The Concept: The Assembly Line

Imagine an assembly line building a product:
- **Each station** checks quality and passes to next station
- **If any station fails**: Stop the line, report which station failed
- **If all succeed**: Final product ready

**The ? operator** lets each function pass along errors automatically.

## The Problem: Error Handling is Verbose

Without the `?` operator, error handling gets messy fast:

```rust
use std::fs;
use std::io;

fn read_and_process_file() -> Result<String, io::Error> {
    let content_result = fs::read_to_string("data.txt");

    let content = match content_result {
        Ok(c) => c,
        Err(e) => return Err(e),
    };

    let trimmed = content.trim();

    let uppercase = trimmed.to_uppercase();

    Ok(uppercase)
}
```

**Problem**: Boilerplate match statements everywhere!

## The Solution: The ? Operator

```rust
use std::fs;
use std::io;

fn read_and_process_file() -> Result<String, io::Error> {
    let content = fs::read_to_string("data.txt")?;  // Auto-return error
    let trimmed = content.trim();
    let uppercase = trimmed.to_uppercase();
    Ok(uppercase)
}
```

**What ? does:**
1. If `Ok(value)`: Unwrap the value, continue
2. If `Err(error)`: Return the error immediately from the function

## How ? Works Behind the Scenes

```rust
// This code:
let value = some_result?;

// Is equivalent to:
let value = match some_result {
    Ok(v) => v,
    Err(e) => return Err(e),
};
```

## Chaining Multiple Operations

```rust
use std::fs;
use std::io;

fn count_lines(filename: &str) -> Result<usize, io::Error> {
    let content = fs::read_to_string(filename)?;  // Might fail
    let lines = content.lines().count();
    Ok(lines)
}

fn main() {
    match count_lines("data.txt") {
        Ok(count) => println!("File has {} lines", count),
        Err(e) => println!("Error: {}", e),
    }
}
```

## Chaining Many ? Operators

```rust
use std::fs;
use std::io;

fn read_username_from_file(path: &str) -> Result<String, io::Error> {
    let content = fs::read_to_string(path)?;  // Error 1: File not found
    let first_line = content
        .lines()
        .next()
        .ok_or(io::Error::new(io::ErrorKind::InvalidData, "Empty file"))?;  // Error 2: Empty file

    Ok(first_line.trim().to_string())
}

fn main() {
    match read_username_from_file("username.txt") {
        Ok(name) => println!("Username: {}", name),
        Err(e) => println!("Error: {}", e),
    }
}
```

## Using ? with Option

The `?` operator also works with `Option<T>`:

```rust
fn get_first_word(text: &str) -> Option<&str> {
    let first = text.split_whitespace().next()?;  // Return None if no words
    Some(first)
}

fn main() {
    let result = get_first_word("Hello world");
    println!("{:?}", result);  // Some("Hello")

    let empty = get_first_word("");
    println!("{:?}", empty);  // None
}
```

## Mixing ? with Different Error Types

**Problem**: Can't use `?` if error types don't match:

```rust
use std::fs;
use std::io;
use std::num::ParseIntError;

// ❌ This won't compile:
// fn read_number_from_file() -> Result<i32, io::Error> {
//     let content = fs::read_to_string("number.txt")?;  // io::Error
//     let number = content.trim().parse::<i32>()?;  // ParseIntError - doesn't match!
//     Ok(number)
// }
```

**Solution 1**: Use a common error type (Box<dyn Error>)

```rust
use std::error::Error;
use std::fs;

fn read_number_from_file() -> Result<i32, Box<dyn Error>> {
    let content = fs::read_to_string("number.txt")?;  // ✅ Works
    let number = content.trim().parse::<i32>()?;       // ✅ Works
    Ok(number)
}

fn main() {
    match read_number_from_file() {
        Ok(num) => println!("Number: {}", num),
        Err(e) => println!("Error: {}", e),
    }
}
```

**Solution 2**: Custom error type (more advanced, see next lesson)

## The ? Operator Rules

### **Rule 1: Function Must Return Result or Option**

```rust
// ❌ Won't compile - main returns ()
// fn main() {
//     let content = std::fs::read_to_string("file.txt")?;  // Error!
// }

// ✅ Works - returns Result
fn read_file() -> Result<String, std::io::Error> {
    let content = std::fs::read_to_string("file.txt")?;
    Ok(content)
}
```

### **Rule 2: Error Types Must Match**

```rust
use std::io;

fn process() -> Result<i32, io::Error> {
    // let num = "42".parse::<i32>()?;  // ❌ ParseIntError doesn't match io::Error
    // Ok(num)

    // Fix: Convert error types
    let num = "42"
        .parse::<i32>()
        .map_err(|e| io::Error::new(io::ErrorKind::InvalidData, e))?;
    Ok(num)
}
```

### **Rule 3: Can't Mix Option and Result**

```rust
// ❌ Can't use Option? in Result function
// fn example() -> Result<i32, String> {
//     let value = Some(42)?;  // Error: returns Option, not Result
//     Ok(value)
// }

// ✅ Convert Option to Result
fn example() -> Result<i32, String> {
    let value = Some(42).ok_or(String::from("No value"))?;
    Ok(value)
}
```

## Making main() Return Result

You can make `main()` return `Result` to use `?`:

```rust
use std::error::Error;
use std::fs;

fn main() -> Result<(), Box<dyn Error>> {
    let content = fs::read_to_string("config.toml")?;  // ✅ Works!
    println!("Config: {}", content);
    Ok(())
}
```

**If error occurs**: Program exits with error message

## Real-World Example: Configuration Loader

```rust
use std::error::Error;
use std::fs;

struct Config {
    port: u16,
    host: String,
}

impl Config {
    fn load(path: &str) -> Result<Self, Box<dyn Error>> {
        let content = fs::read_to_string(path)?;  // Read file

        let lines: Vec<&str> = content.lines().collect();

        // Parse port (line 1)
        let port = lines
            .get(0)
            .ok_or("Missing port")?
            .trim()
            .parse::<u16>()?;

        // Parse host (line 2)
        let host = lines
            .get(1)
            .ok_or("Missing host")?
            .trim()
            .to_string();

        Ok(Config { port, host })
    }
}

fn main() -> Result<(), Box<dyn Error>> {
    let config = Config::load("config.txt")?;
    println!("Server: {}:{}", config.host, config.port);
    Ok(())
}
```

**config.txt:**
```
8080
localhost
```

## Hands-On Practice

```bash
cargo new question_mark_practice
cd question_mark_practice
code .
```

### **Experiment 1: File Reading with ?**

```rust
use std::fs;
use std::io;

fn read_and_uppercase(filename: &str) -> Result<String, io::Error> {
    let content = fs::read_to_string(filename)?;
    Ok(content.to_uppercase())
}

fn main() {
    match read_and_uppercase("test.txt") {
        Ok(content) => println!("{}", content),
        Err(e) => println!("Error: {}", e),
    }
}
```

**Create test.txt first:**
```bash
echo "hello world" > test.txt
```

### **Experiment 2: Parsing Chain**

```rust
use std::error::Error;

fn sum_from_file(filename: &str) -> Result<i32, Box<dyn Error>> {
    let content = std::fs::read_to_string(filename)?;

    let sum = content
        .lines()
        .map(|line| line.trim().parse::<i32>())
        .collect::<Result<Vec<i32>, _>>()?  // Propagate parse errors
        .iter()
        .sum();

    Ok(sum)
}

fn main() -> Result<(), Box<dyn Error>> {
    let total = sum_from_file("numbers.txt")?;
    println!("Total: {}", total);
    Ok(())
}
```

**Create numbers.txt:**
```bash
echo -e "10\n20\n30\n40" > numbers.txt
```

### **Experiment 3: Option to Result Conversion**

```rust
fn get_first_number(numbers: Vec<i32>) -> Result<i32, String> {
    numbers
        .first()
        .copied()
        .ok_or(String::from("Empty list"))  // Convert Option to Result
}

fn main() {
    let nums = vec![1, 2, 3];
    match get_first_number(nums) {
        Ok(n) => println!("First: {}", n),
        Err(e) => println!("Error: {}", e),
    }

    let empty: Vec<i32> = vec![];
    match get_first_number(empty) {
        Ok(n) => println!("First: {}", n),
        Err(e) => println!("Error: {}", e),  // "Empty list"
    }
}
```

### **Challenge: CSV Parser**

Build a simple CSV parser using multiple `?` operators:

```rust
use std::error::Error;
use std::fs;

#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
    city: String,
}

impl Person {
    fn from_csv_line(line: &str) -> Result<Self, Box<dyn Error>> {
        let parts: Vec<&str> = line.split(',').collect();

        let name = parts
            .get(0)
            .ok_or("Missing name")?
            .trim()
            .to_string();

        let age = parts
            .get(1)
            .ok_or("Missing age")?
            .trim()
            .parse::<u32>()?;

        let city = parts
            .get(2)
            .ok_or("Missing city")?
            .trim()
            .to_string();

        Ok(Person { name, age, city })
    }
}

fn load_people(filename: &str) -> Result<Vec<Person>, Box<dyn Error>> {
    let content = fs::read_to_string(filename)?;

    let mut people = Vec::new();

    for (i, line) in content.lines().enumerate() {
        if i == 0 {
            continue;  // Skip header
        }

        let person = Person::from_csv_line(line)
            .map_err(|e| format!("Line {}: {}", i + 1, e))?;

        people.push(person);
    }

    Ok(people)
}

fn main() -> Result<(), Box<dyn Error>> {
    let people = load_people("people.csv")?;

    println!("Loaded {} people:", people.len());
    for person in people {
        println!("{} ({}) from {}", person.name, person.age, person.city);
    }

    Ok(())
}
```

**Create people.csv:**
```csv
name,age,city
Alice,30,New York
Bob,25,San Francisco
Charlie,35,Chicago
```

**Run:**
```bash
cargo run
```

**Output:**
```
Loaded 3 people:
Alice (30) from New York
Bob (25) from San Francisco
Charlie (35) from Chicago
```

## Advanced: Early Return in Loops

```rust
fn find_valid_number(numbers: Vec<&str>) -> Result<i32, String> {
    for (i, num_str) in numbers.iter().enumerate() {
        match num_str.parse::<i32>() {
            Ok(num) => {
                if num > 0 {
                    return Ok(num);  // Found valid number
                }
            }
            Err(_) => {
                return Err(format!("Invalid number at position {}: {}", i, num_str));
            }
        }
    }

    Err(String::from("No positive numbers found"))
}

fn main() {
    let numbers = vec!["0", "-5", "42", "100"];

    match find_valid_number(numbers) {
        Ok(num) => println!("Found: {}", num),
        Err(e) => println!("Error: {}", e),
    }
}
```

## Best Practices

### ✅ DO: Use ? for Clean Code

```rust
fn process() -> Result<String, std::io::Error> {
    let data = std::fs::read_to_string("file.txt")?;
    let trimmed = data.trim();
    Ok(trimmed.to_string())
}
```

### ✅ DO: Make main() Return Result When Needed

```rust
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let config = load_config()?;
    start_server(config)?;
    Ok(())
}
```

### ❌ DON'T: Mix ? with panic!

```rust
// ❌ Bad: Using both error handling strategies
fn bad_example() -> Result<i32, String> {
    let file = std::fs::read_to_string("file.txt")
        .expect("Failed");  // Panics instead of returning Result
    Ok(file.len() as i32)
}

// ✅ Good: Consistent error handling
fn good_example() -> Result<i32, std::io::Error> {
    let file = std::fs::read_to_string("file.txt")?;
    Ok(file.len() as i32)
}
```

### ✅ DO: Add Context to Errors

```rust
use std::fs;

fn load_config() -> Result<String, String> {
    fs::read_to_string("config.toml")
        .map_err(|e| format!("Failed to load config: {}", e))
}
```

## Key Takeaways

- ✅ The `?` operator simplifies error handling dramatically
- ✅ Works with both `Result<T, E>` and `Option<T>`
- ✅ Automatically returns errors early from functions
- ✅ Can only be used in functions returning `Result` or `Option`
- ✅ Error types must match (or use `Box<dyn Error>`)
- ✅ Can chain many `?` operators cleanly
- ✅ `main()` can return `Result<(), Box<dyn Error>>`
- ✅ Use `.ok_or()` to convert `Option` to `Result`
- ✅ Use `.map_err()` to convert between error types
- ✅ Reduces boilerplate while maintaining safety
- ✅ Makes error propagation explicit and compile-time checked

**Next**: Creating custom error types for better error handling!

---

**Progress**: Module 6, Lesson 3 complete (36/60 lessons total)
