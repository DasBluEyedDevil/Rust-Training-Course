# Module 6, Lesson 2: When Things Can Fail with Details — Deep Dive into `Result<T, E>`

## The Concept: The Package Delivery

Imagine ordering a package:
- **Successful delivery**: `Ok(Package)` ✅ - You get your item
- **Failed delivery**: `Err("Address not found")` ❌ - You get an explanation

**Result<T, E>** represents an operation that can succeed with a value or fail with an error.

## Option vs Result: What's the Difference?

```rust
// Option: Something might not exist
fn find_user(id: u32) -> Option<User> {
    // Returns Some(user) or None (no reason given)
}

// Result: Operation might fail (with reason)
fn load_file(path: &str) -> Result<String, std::io::Error> {
    // Returns Ok(contents) or Err(error details)
}
```

**When to use which:**
- **Option**: Value might not exist (no error, just absence)
- **Result**: Operation can fail and you need to know why

## Creating Results

```rust
// Success case
let success: Result<i32, String> = Ok(42);

// Error case
let failure: Result<i32, String> = Err(String::from("Something went wrong"));

// Function returning Result
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Division by zero"))
    } else {
        Ok(a / b)
    }
}
```

## Handling Results: Pattern Matching

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}

fn main() {
    let result = divide(10, 2);

    match result {
        Ok(value) => println!("Result: {}", value),
        Err(error) => println!("Error: {}", error),
    }

    let result2 = divide(10, 0);

    match result2 {
        Ok(value) => println!("Result: {}", value),
        Err(error) => println!("Error: {}", error),  // This prints
    }
}
```

**Output:**
```
Result: 5
Error: Cannot divide by zero
```

## Result Methods

### **is_ok() and is_err() - Quick Checks**

```rust
fn main() {
    let success: Result<i32, String> = Ok(10);
    let failure: Result<i32, String> = Err(String::from("failed"));

    println!("Success is ok: {}", success.is_ok());    // true
    println!("Success is err: {}", success.is_err());  // false

    println!("Failure is ok: {}", failure.is_ok());    // false
    println!("Failure is err: {}", failure.is_err());  // true
}
```

### **unwrap_or() - Provide Default Value**

```rust
fn main() {
    let success: Result<i32, String> = Ok(100);
    let failure: Result<i32, String> = Err(String::from("error"));

    println!("{}", success.unwrap_or(0));  // 100
    println!("{}", failure.unwrap_or(0));  // 0 (default)
}
```

### **unwrap_or_else() - Compute Default from Error**

```rust
fn main() {
    let result: Result<i32, String> = Err(String::from("calculation failed"));

    let value = result.unwrap_or_else(|err| {
        println!("Error occurred: {}", err);
        0  // Return default
    });

    println!("Value: {}", value);
}
```

**Output:**
```
Error occurred: calculation failed
Value: 0
```

### **map() - Transform Success Value**

```rust
fn main() {
    let result: Result<i32, String> = Ok(5);

    let doubled = result.map(|n| n * 2);
    println!("{:?}", doubled);  // Ok(10)

    let failed: Result<i32, String> = Err(String::from("error"));
    let still_failed = failed.map(|n| n * 2);
    println!("{:?}", still_failed);  // Err("error")
}
```

### **map_err() - Transform Error Value**

```rust
fn main() {
    let result: Result<i32, String> = Err(String::from("failed"));

    let new_error = result.map_err(|e| format!("ERROR: {}", e));
    println!("{:?}", new_error);  // Err("ERROR: failed")
}
```

### **and_then() - Chain Operations That Return Results**

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Division by zero"))
    } else {
        Ok(a / b)
    }
}

fn main() {
    let result = divide(10, 2)
        .and_then(|n| divide(n, 0));  // First succeeds, second fails

    match result {
        Ok(val) => println!("Result: {}", val),
        Err(e) => println!("Error: {}", e),  // "Division by zero"
    }
}
```

## Dangerous Methods (Use With Extreme Care!)

### **unwrap() - Panics on Error**

```rust
fn main() {
    let success: Result<i32, String> = Ok(42);
    println!("{}", success.unwrap());  // ✅ Works: 42

    let failure: Result<i32, String> = Err(String::from("failed"));
    // println!("{}", failure.unwrap());  // 💥 PANIC: "called unwrap on Err"
}
```

### **expect() - Panic with Custom Message**

```rust
fn main() {
    let config: Result<String, String> = Err(String::from("file not found"));

    // let content = config.expect("Configuration must be loaded!");
    // 💥 PANIC: "Configuration must be loaded!: file not found"
}
```

### **When to Use unwrap()/expect()**

Only in these situations:
1. **Prototyping**: Quick testing before proper error handling
2. **Tests**: Where panics are acceptable
3. **When impossible to fail**: But add comment explaining why

```rust
fn main() {
    // Safe: parsing a string literal we control
    let number = "42".parse::<i32>().expect("hardcoded string is valid");
    println!("{}", number);
}
```

## The ? Operator - Early Return on Error

The `?` operator is **game-changing** for clean error handling:

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = fs::read_to_string("username.txt")?;
    // If error, immediately return Err(error)
    // If success, unwrap and continue

    username = username.trim().to_string();
    Ok(username)
}

fn main() {
    match read_username_from_file() {
        Ok(name) => println!("Username: {}", name),
        Err(e) => println!("Error reading file: {}", e),
    }
}
```

**How ? works:**
- If `Ok(value)`: Extract value and continue
- If `Err(error)`: Return error immediately from function

**Without ? operator** (verbose):
```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let result = fs::read_to_string("username.txt");

    let mut username = match result {
        Ok(content) => content,
        Err(e) => return Err(e),
    };

    username = username.trim().to_string();
    Ok(username)
}
```

**With ? operator** (clean):
```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = fs::read_to_string("username.txt")?;
    username = username.trim().to_string();
    Ok(username)
}
```

## Chaining with ?

```rust
use std::fs;
use std::io;

fn process_file(path: &str) -> Result<usize, io::Error> {
    let content = fs::read_to_string(path)?;  // Return error if fails
    let trimmed = content.trim();
    let length = trimmed.len();
    Ok(length)
}

fn main() {
    match process_file("data.txt") {
        Ok(len) => println!("File length: {}", len),
        Err(e) => println!("Error: {}", e),
    }
}
```

## Real-World Example: User Input Validation

```rust
use std::io;

#[derive(Debug)]
enum ValidationError {
    TooShort,
    TooLong,
    InvalidCharacters,
}

fn validate_username(name: &str) -> Result<String, ValidationError> {
    if name.len() < 3 {
        return Err(ValidationError::TooShort);
    }

    if name.len() > 20 {
        return Err(ValidationError::TooLong);
    }

    if !name.chars().all(|c| c.is_alphanumeric() || c == '_') {
        return Err(ValidationError::InvalidCharacters);
    }

    Ok(name.to_string())
}

fn main() {
    println!("Enter username:");
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to read line");

    let username = input.trim();

    match validate_username(username) {
        Ok(valid_name) => println!("✅ Valid username: {}", valid_name),
        Err(ValidationError::TooShort) => {
            println!("❌ Username must be at least 3 characters");
        }
        Err(ValidationError::TooLong) => {
            println!("❌ Username must be at most 20 characters");
        }
        Err(ValidationError::InvalidCharacters) => {
            println!("❌ Username can only contain letters, numbers, and underscores");
        }
    }
}
```

## Hands-On Practice

```bash
cargo new result_practice
cd result_practice
code .
```

### **Experiment 1: Safe Parsing**

```rust
fn parse_age(input: &str) -> Result<u32, String> {
    match input.parse::<u32>() {
        Ok(age) => {
            if age > 0 && age < 150 {
                Ok(age)
            } else {
                Err(String::from("Age must be between 1 and 149"))
            }
        }
        Err(_) => Err(String::from("Invalid number format")),
    }
}

fn main() {
    let test_cases = vec!["25", "200", "abc", "0"];

    for input in test_cases {
        match parse_age(input) {
            Ok(age) => println!("✅ Valid age: {}", age),
            Err(e) => println!("❌ Error for '{}': {}", input, e),
        }
    }
}
```

### **Experiment 2: Multiple Validations**

```rust
fn validate_email(email: &str) -> Result<String, String> {
    if !email.contains('@') {
        return Err(String::from("Must contain @"));
    }

    if !email.contains('.') {
        return Err(String::from("Must contain domain"));
    }

    if email.len() < 5 {
        return Err(String::from("Too short"));
    }

    Ok(email.to_string())
}

fn main() {
    let emails = vec![
        "user@example.com",
        "invalid",
        "no-at-sign.com",
        "x@y",
    ];

    for email in emails {
        match validate_email(email) {
            Ok(valid) => println!("✅ {}", valid),
            Err(e) => println!("❌ {}: {}", email, e),
        }
    }
}
```

### **Experiment 3: Using ? Operator**

```rust
fn calculate_average(numbers: Vec<&str>) -> Result<f64, String> {
    if numbers.is_empty() {
        return Err(String::from("Cannot calculate average of empty list"));
    }

    let mut sum = 0.0;
    let mut count = 0;

    for num_str in numbers {
        let num = num_str
            .parse::<f64>()
            .map_err(|_| format!("Invalid number: {}", num_str))?;

        sum += num;
        count += 1;
    }

    Ok(sum / count as f64)
}

fn main() {
    let valid = vec!["10", "20", "30"];
    match calculate_average(valid) {
        Ok(avg) => println!("Average: {:.2}", avg),
        Err(e) => println!("Error: {}", e),
    }

    let invalid = vec!["10", "abc", "30"];
    match calculate_average(invalid) {
        Ok(avg) => println!("Average: {:.2}", avg),
        Err(e) => println!("Error: {}", e),
    }
}
```

### **Challenge: Password Validator**

Build a comprehensive password validation system:

```rust
#[derive(Debug)]
enum PasswordError {
    TooShort,
    NoUppercase,
    NoLowercase,
    NoDigit,
    NoSpecialChar,
}

fn validate_password(password: &str) -> Result<String, Vec<PasswordError>> {
    let mut errors = Vec::new();

    if password.len() < 8 {
        errors.push(PasswordError::TooShort);
    }

    if !password.chars().any(|c| c.is_uppercase()) {
        errors.push(PasswordError::NoUppercase);
    }

    if !password.chars().any(|c| c.is_lowercase()) {
        errors.push(PasswordError::NoLowercase);
    }

    if !password.chars().any(|c| c.is_numeric()) {
        errors.push(PasswordError::NoDigit);
    }

    if !password.chars().any(|c| "!@#$%^&*".contains(c)) {
        errors.push(PasswordError::NoSpecialChar);
    }

    if errors.is_empty() {
        Ok(password.to_string())
    } else {
        Err(errors)
    }
}

fn main() {
    let test_passwords = vec![
        "Abc123!@",
        "password",
        "PASSWORD",
        "Pass123",
        "short",
        "MyP@ssw0rd",
    ];

    for pwd in test_passwords {
        print!("Password '{}': ", pwd);

        match validate_password(pwd) {
            Ok(_) => println!("✅ Valid!"),
            Err(errors) => {
                println!("❌ Invalid:");
                for error in errors {
                    let msg = match error {
                        PasswordError::TooShort => "  - Must be at least 8 characters",
                        PasswordError::NoUppercase => "  - Must contain uppercase letter",
                        PasswordError::NoLowercase => "  - Must contain lowercase letter",
                        PasswordError::NoDigit => "  - Must contain a digit",
                        PasswordError::NoSpecialChar => "  - Must contain special character (!@#$%^&*)",
                    };
                    println!("{}", msg);
                }
            }
        }
        println!();
    }
}
```

**Output:**
```
Password 'Abc123!@': ✅ Valid!

Password 'password': ❌ Invalid:
  - Must contain uppercase letter
  - Must contain a digit
  - Must contain special character (!@#$%^&*)

Password 'MyP@ssw0rd': ✅ Valid!
```

## Converting Between Option and Result

```rust
fn main() {
    // Option to Result
    let maybe_number: Option<i32> = Some(42);
    let result = maybe_number.ok_or(String::from("No value"));
    println!("{:?}", result);  // Ok(42)

    // Result to Option
    let result: Result<i32, String> = Ok(100);
    let option = result.ok();
    println!("{:?}", option);  // Some(100)

    // Error to Option (discards error info)
    let result: Result<i32, String> = Err(String::from("failed"));
    let option = result.ok();
    println!("{:?}", option);  // None
}
```

## Key Takeaways

- ✅ `Result<T, E>` represents operations that can fail with error details
- ✅ Two variants: `Ok(value)` for success, `Err(error)` for failure
- ✅ Use `match` or `if let` to handle both cases
- ✅ `.unwrap_or(default)` provides safe fallbacks
- ✅ `.map()` transforms success values
- ✅ `.map_err()` transforms error values
- ✅ `.and_then()` chains Result-returning operations
- ✅ The `?` operator enables clean error propagation
- ✅ Avoid `.unwrap()` in production code
- ✅ Use `.expect()` with descriptive messages
- ✅ Custom enums make excellent error types
- ✅ Use `Result` when you need error details, `Option` when you just need presence/absence

**Next**: The ? operator in depth and error propagation patterns!

---

**Progress**: Module 6, Lesson 2 complete (35/60 lessons total)
