# Module 6, Lesson 1: When Things Might Not Be There — Deep Dive into `Option<T>`

## The Concept: The Vending Machine

Imagine a vending machine:
- **Press button for Snickers**: Machine dispenses `Some(Snickers)` ✅
- **Press button for sold-out item**: Machine dispenses `None` ❌
- **You always get a response**: Either something or nothing

**Option<T>** represents a value that might or might not exist.

## Why Option<T> Exists

In many languages, "nothing" is represented by `null` or `nil`:

```javascript
// JavaScript - The Billion Dollar Mistake
let user = findUser("alice");
console.log(user.name);  // 💥 Runtime error if user is null!
```

**Rust's Solution**: Make "might not exist" explicit in the type system.

```rust
fn find_user(name: &str) -> Option<User> {
    // Returns Some(user) or None
}
```

The compiler **forces** you to handle the None case!

## Creating Options

```rust
// Some with a value
let some_number = Some(5);
let some_string = Some(String::from("hello"));

// None (no value)
let no_number: Option<i32> = None;

// Function returning Option
fn get_first_element(list: Vec<i32>) -> Option<i32> {
    if list.is_empty() {
        None
    } else {
        Some(list[0])
    }
}
```

## Checking What's Inside

### **Method 1: Pattern Matching** (Most Powerful)

```rust
fn main() {
    let maybe_number = Some(42);

    match maybe_number {
        Some(num) => println!("Got number: {}", num),
        None => println!("Got nothing"),
    }
}
```

### **Method 2: if let** (More Concise)

```rust
fn main() {
    let maybe_name = Some("Alice");

    if let Some(name) = maybe_name {
        println!("Hello, {}!", name);
    } else {
        println!("No name provided");
    }
}
```

### **Method 3: Checking Without Extracting**

```rust
fn main() {
    let x = Some(10);

    if x.is_some() {
        println!("x has a value");
    }

    if x.is_none() {
        println!("x is empty");
    }
}
```

## Powerful Option Methods

### **unwrap_or() - Provide a Default**

```rust
fn main() {
    let some_value = Some(100);
    let no_value: Option<i32> = None;

    println!("{}", some_value.unwrap_or(0));  // 100
    println!("{}", no_value.unwrap_or(0));    // 0 (default)
}
```

### **map() - Transform the Value Inside**

```rust
fn main() {
    let maybe_number = Some(5);

    // Multiply the number inside by 2
    let doubled = maybe_number.map(|n| n * 2);

    println!("{:?}", doubled);  // Some(10)

    // If None, stays None
    let nothing: Option<i32> = None;
    let still_nothing = nothing.map(|n| n * 2);
    println!("{:?}", still_nothing);  // None
}
```

### **and_then() - Chain Operations That Return Options**

```rust
fn divide(numerator: i32, denominator: i32) -> Option<i32> {
    if denominator == 0 {
        None
    } else {
        Some(numerator / denominator)
    }
}

fn main() {
    let result = Some(10)
        .and_then(|n| divide(n, 2))  // 10 / 2 = Some(5)
        .and_then(|n| divide(n, 0)); // 5 / 0 = None

    match result {
        Some(val) => println!("Result: {}", val),
        None => println!("Division by zero!"),
    }
}
```

### **filter() - Keep Only If Condition Met**

```rust
fn main() {
    let number = Some(42);

    // Keep only if even
    let even = number.filter(|n| n % 2 == 0);
    println!("{:?}", even);  // Some(42)

    // Filter out odd numbers
    let odd_number = Some(43);
    let filtered = odd_number.filter(|n| n % 2 == 0);
    println!("{:?}", filtered);  // None
}
```

### **or() and or_else() - Provide Alternatives**

```rust
fn main() {
    let primary = None;
    let backup = Some(100);

    let result = primary.or(backup);
    println!("{:?}", result);  // Some(100)

    // or_else with computation
    let result = primary.or_else(|| Some(42));
    println!("{:?}", result);  // Some(42)
}
```

## Dangerous Methods (Use With Care!)

### **unwrap() - Panics if None**

```rust
fn main() {
    let some_value = Some(10);
    println!("{}", some_value.unwrap());  // ✅ Works: 10

    let no_value: Option<i32> = None;
    // println!("{}", no_value.unwrap());  // 💥 PANIC! "called unwrap on None"
}
```

**Rule**: Only use `unwrap()` when you're **absolutely certain** there's a value.

### **expect() - Panic with Custom Message**

```rust
fn main() {
    let config = Some("settings.toml");
    let filename = config.expect("Config file must be provided!");
    println!("{}", filename);

    let missing: Option<&str> = None;
    // let file = missing.expect("Config file must be provided!");
    // 💥 PANIC: "Config file must be provided!"
}
```

## Real-World Example: User Lookup

```rust
use std::collections::HashMap;

struct User {
    name: String,
    email: String,
    age: u32,
}

fn find_user(users: &HashMap<u32, User>, id: u32) -> Option<&User> {
    users.get(&id)
}

fn main() {
    let mut users = HashMap::new();

    users.insert(
        1,
        User {
            name: String::from("Alice"),
            email: String::from("alice@example.com"),
            age: 30,
        },
    );

    // Found user
    match find_user(&users, 1) {
        Some(user) => println!("Found: {} ({})", user.name, user.email),
        None => println!("User not found"),
    }

    // Missing user
    match find_user(&users, 999) {
        Some(user) => println!("Found: {}", user.name),
        None => println!("User not found"),  // This prints
    }
}
```

## Hands-On Practice

```bash
cargo new option_practice
cd option_practice
code .
```

### **Experiment 1: Safe Division**

```rust
fn safe_divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None
    } else {
        Some(a / b)
    }
}

fn main() {
    let result1 = safe_divide(10.0, 2.0);
    println!("{:?}", result1);  // Some(5.0)

    let result2 = safe_divide(10.0, 0.0);
    println!("{:?}", result2);  // None

    // With unwrap_or
    let safe_result = safe_divide(10.0, 0.0).unwrap_or(0.0);
    println!("Safe result: {}", safe_result);  // 0.0
}
```

### **Experiment 2: Parsing Numbers**

```rust
fn main() {
    let valid = "42";
    let invalid = "hello";

    // parse returns Option<i32>
    let num1: Option<i32> = valid.parse().ok();
    let num2: Option<i32> = invalid.parse().ok();

    println!("{:?}", num1);  // Some(42)
    println!("{:?}", num2);  // None

    // With default
    let parsed = num2.unwrap_or(0);
    println!("Parsed: {}", parsed);  // 0
}
```

### **Experiment 3: Chaining with map()**

```rust
fn main() {
    let user_input = Some("  hello world  ");

    let processed = user_input
        .map(|s| s.trim())           // Remove whitespace
        .map(|s| s.to_uppercase())   // Convert to uppercase
        .filter(|s| s.len() > 5);    // Keep only if > 5 chars

    println!("{:?}", processed);  // Some("HELLO WORLD")
}
```

### **Challenge: Configuration Parser**

Build a config system that safely handles missing values:

```rust
use std::collections::HashMap;

struct Config {
    settings: HashMap<String, String>,
}

impl Config {
    fn new() -> Self {
        Config {
            settings: HashMap::new(),
        }
    }

    fn set(&mut self, key: &str, value: &str) {
        self.settings.insert(key.to_string(), value.to_string());
    }

    fn get(&self, key: &str) -> Option<&String> {
        self.settings.get(key)
    }

    fn get_or(&self, key: &str, default: &str) -> String {
        self.get(key)
            .map(|s| s.clone())
            .unwrap_or_else(|| default.to_string())
    }

    fn get_as_int(&self, key: &str) -> Option<i32> {
        self.get(key)
            .and_then(|s| s.parse::<i32>().ok())
    }
}

fn main() {
    let mut config = Config::new();

    config.set("port", "8080");
    config.set("host", "localhost");
    config.set("debug", "true");

    // Get existing value
    if let Some(port) = config.get("port") {
        println!("Port: {}", port);
    }

    // Get missing value with default
    let timeout = config.get_or("timeout", "30");
    println!("Timeout: {}", timeout);

    // Parse as integer
    match config.get_as_int("port") {
        Some(num) => println!("Port as number: {}", num),
        None => println!("Port is not a valid number"),
    }

    // Missing value
    match config.get_as_int("missing") {
        Some(num) => println!("Value: {}", num),
        None => println!("Key not found"),  // This prints
    }
}
```

**Output:**
```
Port: 8080
Timeout: 30
Port as number: 8080
Key not found
```

## Common Patterns

### **Pattern 1: Early Return with ?**

```rust
fn get_first_char(text: Option<String>) -> Option<char> {
    let t = text?;  // Return None if text is None
    t.chars().next()  // Return first char or None
}

fn main() {
    let result1 = get_first_char(Some(String::from("hello")));
    println!("{:?}", result1);  // Some('h')

    let result2 = get_first_char(None);
    println!("{:?}", result2);  // None
}
```

### **Pattern 2: Combining Multiple Options**

```rust
fn add_optional_numbers(a: Option<i32>, b: Option<i32>) -> Option<i32> {
    match (a, b) {
        (Some(x), Some(y)) => Some(x + y),
        _ => None,
    }
}

fn main() {
    println!("{:?}", add_optional_numbers(Some(5), Some(3)));  // Some(8)
    println!("{:?}", add_optional_numbers(Some(5), None));     // None
}
```

### **Pattern 3: Optional References**

```rust
fn find_longest<'a>(words: &'a [String]) -> Option<&'a String> {
    if words.is_empty() {
        None
    } else {
        Some(&words[0])  // Simplified - just return first
    }
}

fn main() {
    let words = vec![
        String::from("hello"),
        String::from("world"),
    ];

    match find_longest(&words) {
        Some(word) => println!("Found: {}", word),
        None => println!("Empty list"),
    }
}
```

## Key Takeaways

- ✅ `Option<T>` represents a value that might not exist
- ✅ Two variants: `Some(value)` and `None`
- ✅ Forces you to handle the "missing" case at compile time
- ✅ Use pattern matching or `if let` to extract values
- ✅ `.unwrap_or(default)` provides safe defaults
- ✅ `.map()` transforms the value inside
- ✅ `.and_then()` chains operations that return Options
- ✅ `.filter()` conditionally keeps values
- ✅ Avoid `.unwrap()` unless you're **certain** there's a value
- ✅ Use `.expect()` with helpful messages for debugging
- ✅ The `?` operator allows early returns from functions

**Next**: Deep dive into `Result<T, E>` for operations that can fail with error information!

---

**Progress**: Module 6, Lesson 1 complete (34/60 lessons total)
