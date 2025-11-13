# Module 3, Lesson 4: When Things Can Go Wrong — The `Result` Enum

## The Concept: Success or Failure

Imagine trying to open a file on your computer. There are two possible outcomes:
- **Success** — The file exists and you can read it
- **Failure** — The file doesn't exist, or you don't have permission

Unlike `Option` (which represents "something or nothing"), `Result` represents "success or failure" and can tell you WHY it failed.

## The `Result` Enum

`Result` is a built-in enum with two variants:

```rust
enum Result<T, E> {
    Ok(T),      // Success with a value of type T
    Err(E),     // Error with an error value of type E
}
```

**Examples:**

```rust
let success: Result<i32, String> = Ok(42);
let failure: Result<i32, String> = Err(String::from("Something went wrong"));
```

## Why Use Result?

**The problem in other languages:**

```python
# Python example
try:
    file = open("data.txt")
except:
    # What error occurred? File not found? Permission denied?
    pass
```

**The Rust way:**

```rust
use std::fs::File;

fn open_file() -> Result<File, std::io::Error> {
    File::open("data.txt")  // Returns Ok(file) or Err(error)
}

fn main() {
    match open_file() {
        Ok(file) => println!("File opened successfully!"),
        Err(error) => println!("Error: {}", error),
    }
}
```

The error contains specific information about what went wrong!

## Using Result with match

```rust
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}

fn main() {
    let result1 = divide(10.0, 2.0);
    let result2 = divide(10.0, 0.0);

    match result1 {
        Ok(value) => println!("Result: {}", value),
        Err(error) => println!("Error: {}", error),
    }

    match result2 {
        Ok(value) => println!("Result: {}", value),
        Err(error) => println!("Error: {}", error),
    }
}
```

Output:
```
Result: 5
Error: Cannot divide by zero
```

## Unwrapping Results

**`.unwrap()` — Get the value or panic:**

```rust
fn main() {
    let result = divide(10.0, 2.0);
    let value = result.unwrap();  // Gets 5.0
    println!("{}", value);

    let bad_result = divide(10.0, 0.0);
    let value = bad_result.unwrap();  // PANICS! Program crashes
}
```

⚠️ **Only use `.unwrap()` when you're CERTAIN there's no error!**

**`.expect()` — Like unwrap but with a custom message:**

```rust
fn main() {
    let result = divide(10.0, 0.0);
    let value = result.expect("Division should never fail here");  // Panics with custom message
}
```

**`.unwrap_or()` — Provide a default value:**

```rust
fn main() {
    let result = divide(10.0, 0.0);
    let value = result.unwrap_or(0.0);  // Returns 0.0 if error
    println!("{}", value);
}
```

## The `?` Operator (Propagating Errors)

When a function returns `Result`, you can propagate errors upward with `?`:

```rust
fn parse_number(s: &str) -> Result<i32, String> {
    match s.parse::<i32>() {
        Ok(num) => Ok(num),
        Err(_) => Err(String::from("Not a valid number")),
    }
}

fn add_numbers(a: &str, b: &str) -> Result<i32, String> {
    let num1 = parse_number(a)?;  // If error, return early
    let num2 = parse_number(b)?;  // If error, return early
    Ok(num1 + num2)
}

fn main() {
    match add_numbers("10", "20") {
        Ok(sum) => println!("Sum: {}", sum),
        Err(e) => println!("Error: {}", e),
    }

    match add_numbers("10", "abc") {
        Ok(sum) => println!("Sum: {}", sum),
        Err(e) => println!("Error: {}", e),
    }
}
```

The `?` operator means: "If this is an error, return it immediately. Otherwise, unwrap the Ok value."

We'll explore `?` more in Module 6!

## if let with Result

Handle only the success case:

```rust
fn main() {
    let result = divide(10.0, 2.0);

    if let Ok(value) = result {
        println!("Result: {}", value);
    }
    // Errors are silently ignored
}
```

Or only the error case:

```rust
fn main() {
    let result = divide(10.0, 0.0);

    if let Err(error) = result {
        println!("Error occurred: {}", error);
    }
}
```

## Practical Examples

**Parsing user input:**

```rust
fn parse_age(input: &str) -> Result<u32, String> {
    match input.trim().parse::<u32>() {
        Ok(age) if age > 0 && age < 150 => Ok(age),
        Ok(_) => Err(String::from("Age out of valid range")),
        Err(_) => Err(String::from("Not a valid number")),
    }
}

fn main() {
    let inputs = ["25", "200", "abc", "0"];

    for input in inputs {
        match parse_age(input) {
            Ok(age) => println!("{} is a valid age", age),
            Err(e) => println!("'{}' - Error: {}", input, e),
        }
    }
}
```

**File operations:**

```rust
use std::fs;

fn read_username_from_file() -> Result<String, String> {
    match fs::read_to_string("username.txt") {
        Ok(contents) => Ok(contents.trim().to_string()),
        Err(error) => Err(format!("Could not read file: {}", error)),
    }
}

fn main() {
    match read_username_from_file() {
        Ok(username) => println!("Username: {}", username),
        Err(error) => println!("Error: {}", error),
    }
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new result_practice
cd result_practice
code .
```

### **Experiment 1: Basic Result**

```rust
fn sqrt(n: f64) -> Result<f64, String> {
    if n < 0.0 {
        Err(String::from("Cannot take square root of negative number"))
    } else {
        Ok(n.sqrt())
    }
}

fn main() {
    let numbers = [16.0, -4.0, 9.0];

    for num in numbers {
        match sqrt(num) {
            Ok(result) => println!("sqrt({}) = {}", num, result),
            Err(e) => println!("sqrt({}) - Error: {}", num, e),
        }
    }
}
```

### **Experiment 2: Parsing Input**

```rust
fn parse_and_double(s: &str) -> Result<i32, String> {
    match s.parse::<i32>() {
        Ok(num) => Ok(num * 2),
        Err(_) => Err(format!("'{}' is not a valid number", s)),
    }
}

fn main() {
    let inputs = ["42", "abc", "100", "xyz"];

    for input in inputs {
        match parse_and_double(input) {
            Ok(result) => println!("{} * 2 = {}", input, result),
            Err(e) => println!("Error: {}", e),
        }
    }
}
```

### **Challenge: Temperature Converter with Validation**

Create a function that converts Celsius to Fahrenheit, but returns an error if the temperature is below absolute zero (-273.15°C):

```rust
fn celsius_to_fahrenheit(celsius: f64) -> Result<f64, String> {
    if celsius < -273.15 {
        Err(String::from("Temperature below absolute zero!"))
    } else {
        Ok(celsius * 1.8 + 32.0)
    }
}

fn main() {
    let temperatures = [25.0, -300.0, 0.0, -273.15];

    for temp in temperatures {
        match celsius_to_fahrenheit(temp) {
            Ok(f) => println!("{}°C = {}°F", temp, f),
            Err(e) => println!("{}°C - Error: {}", temp, e),
        }
    }
}
```

## Key Takeaways

- ✅ `Result<T, E>` represents success (`Ok`) or failure (`Err`)
- ✅ `Ok(value)` contains the successful result
- ✅ `Err(error)` contains information about the error
- ✅ Use `match` to handle both cases
- ✅ `.unwrap()` gets the value but panics on error
- ✅ `.unwrap_or(default)` provides a fallback value
- ✅ `.expect(msg)` like unwrap with custom panic message
- ✅ `?` operator propagates errors (more in Module 6!)
- ✅ More informative than `Option` when errors need context
