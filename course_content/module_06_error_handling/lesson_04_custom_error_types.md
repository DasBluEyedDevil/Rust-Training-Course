# Module 6, Lesson 4: Making Errors Meaningful — Custom Error Types

## The Concept: Specific Doctor Diagnoses

Imagine visiting a doctor:
- **Generic**: "You're sick" ❌ (not helpful)
- **Specific**: "You have strep throat, here's the treatment" ✅ (actionable)

**Custom error types** provide specific, actionable error information.

## The Problem with String Errors

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}
```

**Problems:**
- Can't match on specific error types
- Hard to handle different errors differently
- No structured error data
- Strings aren't very Rust-idiomatic

## Solution: Custom Error Enums

```rust
#[derive(Debug)]
enum MathError {
    DivisionByZero,
    Overflow,
    NegativeResult,
}

fn divide(a: i32, b: i32) -> Result<i32, MathError> {
    if b == 0 {
        return Err(MathError::DivisionByZero);
    }

    if a == i32::MAX && b == -1 {
        return Err(MathError::Overflow);
    }

    let result = a / b;

    if result < 0 {
        return Err(MathError::NegativeResult);
    }

    Ok(result)
}

fn main() {
    match divide(10, 0) {
        Ok(result) => println!("Result: {}", result),
        Err(MathError::DivisionByZero) => println!("❌ Cannot divide by zero!"),
        Err(MathError::Overflow) => println!("❌ Result too large!"),
        Err(MathError::NegativeResult) => println!("❌ Negative results not allowed!"),
    }
}
```

**Benefits:**
- ✅ Pattern match on specific errors
- ✅ Type-safe error handling
- ✅ Self-documenting code
- ✅ Compiler ensures all errors handled

## Adding Data to Errors

```rust
#[derive(Debug)]
enum ValidationError {
    TooShort { min_length: usize, actual: usize },
    TooLong { max_length: usize, actual: usize },
    InvalidCharacter(char),
    Empty,
}

fn validate_username(name: &str) -> Result<String, ValidationError> {
    if name.is_empty() {
        return Err(ValidationError::Empty);
    }

    if name.len() < 3 {
        return Err(ValidationError::TooShort {
            min_length: 3,
            actual: name.len(),
        });
    }

    if name.len() > 20 {
        return Err(ValidationError::TooLong {
            max_length: 20,
            actual: name.len(),
        });
    }

    for ch in name.chars() {
        if !ch.is_alphanumeric() && ch != '_' {
            return Err(ValidationError::InvalidCharacter(ch));
        }
    }

    Ok(name.to_string())
}

fn main() {
    let usernames = vec!["alice", "ab", "user@name", "this_is_way_too_long_username"];

    for name in usernames {
        match validate_username(name) {
            Ok(valid) => println!("✅ '{}' is valid", valid),
            Err(ValidationError::Empty) => {
                println!("❌ '{}': Username cannot be empty", name);
            }
            Err(ValidationError::TooShort { min_length, actual }) => {
                println!(
                    "❌ '{}': Too short (min: {}, got: {})",
                    name, min_length, actual
                );
            }
            Err(ValidationError::TooLong { max_length, actual }) => {
                println!(
                    "❌ '{}': Too long (max: {}, got: {})",
                    name, max_length, actual
                );
            }
            Err(ValidationError::InvalidCharacter(ch)) => {
                println!("❌ '{}': Invalid character '{}'", name, ch);
            }
        }
    }
}
```

**Output:**
```
✅ 'alice' is valid
❌ 'ab': Too short (min: 3, got: 2)
❌ 'user@name': Invalid character '@'
❌ 'this_is_way_too_long_username': Too long (max: 20, got: 29)
```

## Implementing Display for Better Messages

```rust
use std::fmt;

#[derive(Debug)]
enum FileError {
    NotFound(String),
    PermissionDenied(String),
    AlreadyExists(String),
}

impl fmt::Display for FileError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            FileError::NotFound(path) => {
                write!(f, "File not found: {}", path)
            }
            FileError::PermissionDenied(path) => {
                write!(f, "Permission denied: {}", path)
            }
            FileError::AlreadyExists(path) => {
                write!(f, "File already exists: {}", path)
            }
        }
    }
}

fn open_file(path: &str) -> Result<String, FileError> {
    // Simulate different errors
    if path.is_empty() {
        return Err(FileError::NotFound(String::from("(empty path)")));
    }

    if path.starts_with("/system") {
        return Err(FileError::PermissionDenied(path.to_string()));
    }

    Ok(format!("Contents of {}", path))
}

fn main() {
    match open_file("") {
        Ok(content) => println!("{}", content),
        Err(e) => println!("Error: {}", e),  // Uses Display trait
    }

    match open_file("/system/secret.txt") {
        Ok(content) => println!("{}", content),
        Err(e) => println!("Error: {}", e),
    }
}
```

**Output:**
```
Error: File not found: (empty path)
Error: Permission denied: /system/secret.txt
```

## Implementing the Error Trait

To make custom errors work with `Box<dyn Error>`:

```rust
use std::error::Error;
use std::fmt;

#[derive(Debug)]
enum DatabaseError {
    ConnectionFailed,
    QueryError(String),
    NotFound,
}

impl fmt::Display for DatabaseError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            DatabaseError::ConnectionFailed => write!(f, "Failed to connect to database"),
            DatabaseError::QueryError(msg) => write!(f, "Query error: {}", msg),
            DatabaseError::NotFound => write!(f, "Record not found"),
        }
    }
}

impl Error for DatabaseError {}

fn fetch_user(id: u32) -> Result<String, DatabaseError> {
    if id == 0 {
        return Err(DatabaseError::NotFound);
    }

    Ok(format!("User {}", id))
}

fn main() -> Result<(), Box<dyn Error>> {
    let user = fetch_user(0)?;  // Works with ? operator now!
    println!("{}", user);
    Ok(())
}
```

## Wrapping Other Errors

```rust
use std::error::Error;
use std::fmt;
use std::fs;
use std::io;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    IoError(io::Error),
    ParseError(ParseIntError),
    InvalidData(String),
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::IoError(e) => write!(f, "IO error: {}", e),
            AppError::ParseError(e) => write!(f, "Parse error: {}", e),
            AppError::InvalidData(msg) => write!(f, "Invalid data: {}", msg),
        }
    }
}

impl Error for AppError {}

impl From<io::Error> for AppError {
    fn from(error: io::Error) -> Self {
        AppError::IoError(error)
    }
}

impl From<ParseIntError> for AppError {
    fn from(error: ParseIntError) -> Self {
        AppError::ParseError(error)
    }
}

fn read_number(filename: &str) -> Result<i32, AppError> {
    let content = fs::read_to_string(filename)?;  // io::Error auto-converted
    let number = content.trim().parse::<i32>()?;  // ParseIntError auto-converted

    if number < 0 {
        return Err(AppError::InvalidData(String::from("Number must be positive")));
    }

    Ok(number)
}

fn main() {
    match read_number("number.txt") {
        Ok(num) => println!("Number: {}", num),
        Err(e) => println!("Error: {}", e),
    }
}
```

**The `From` trait enables automatic error conversion with `?`!**

## Real-World Example: User Registration System

```rust
use std::error::Error;
use std::fmt;

#[derive(Debug)]
enum RegistrationError {
    UsernameTooShort,
    UsernameTooLong,
    UsernameInvalidChars,
    PasswordTooWeak,
    EmailInvalid,
    UsernameExists(String),
}

impl fmt::Display for RegistrationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            RegistrationError::UsernameTooShort => {
                write!(f, "Username must be at least 3 characters")
            }
            RegistrationError::UsernameTooLong => {
                write!(f, "Username must be at most 20 characters")
            }
            RegistrationError::UsernameInvalidChars => {
                write!(f, "Username can only contain letters, numbers, and underscores")
            }
            RegistrationError::PasswordTooWeak => {
                write!(f, "Password must be at least 8 characters and contain letters and numbers")
            }
            RegistrationError::EmailInvalid => {
                write!(f, "Email address is invalid")
            }
            RegistrationError::UsernameExists(name) => {
                write!(f, "Username '{}' already exists", name)
            }
        }
    }
}

impl Error for RegistrationError {}

struct User {
    username: String,
    email: String,
}

fn validate_username(username: &str, existing: &[String]) -> Result<(), RegistrationError> {
    if username.len() < 3 {
        return Err(RegistrationError::UsernameTooShort);
    }

    if username.len() > 20 {
        return Err(RegistrationError::UsernameTooLong);
    }

    if !username.chars().all(|c| c.is_alphanumeric() || c == '_') {
        return Err(RegistrationError::UsernameInvalidChars);
    }

    if existing.contains(&username.to_string()) {
        return Err(RegistrationError::UsernameExists(username.to_string()));
    }

    Ok(())
}

fn validate_password(password: &str) -> Result<(), RegistrationError> {
    let has_letter = password.chars().any(|c| c.is_alphabetic());
    let has_number = password.chars().any(|c| c.is_numeric());

    if password.len() < 8 || !has_letter || !has_number {
        return Err(RegistrationError::PasswordTooWeak);
    }

    Ok(())
}

fn validate_email(email: &str) -> Result<(), RegistrationError> {
    if !email.contains('@') || !email.contains('.') {
        return Err(RegistrationError::EmailInvalid);
    }

    Ok(())
}

fn register_user(
    username: &str,
    password: &str,
    email: &str,
    existing_users: &[String],
) -> Result<User, RegistrationError> {
    validate_username(username, existing_users)?;
    validate_password(password)?;
    validate_email(email)?;

    Ok(User {
        username: username.to_string(),
        email: email.to_string(),
    })
}

fn main() {
    let existing = vec![String::from("alice"), String::from("bob")];

    let test_cases = vec![
        ("alice", "Password123", "alice@example.com"),
        ("ab", "Password123", "user@example.com"),
        ("newuser", "weak", "user@example.com"),
        ("new_user", "Password123", "invalid-email"),
        ("charlie", "Password123", "charlie@example.com"),
    ];

    for (username, password, email) in test_cases {
        println!("\nTrying to register '{}'...", username);

        match register_user(username, password, email, &existing) {
            Ok(user) => {
                println!("✅ Success! Registered {} with email {}", user.username, user.email);
            }
            Err(e) => {
                println!("❌ Registration failed: {}", e);
            }
        }
    }
}
```

**Output:**
```
Trying to register 'alice'...
❌ Registration failed: Username 'alice' already exists

Trying to register 'ab'...
❌ Registration failed: Username must be at least 3 characters

Trying to register 'newuser'...
❌ Registration failed: Password must be at least 8 characters and contain letters and numbers

Trying to register 'new_user'...
❌ Registration failed: Email address is invalid

Trying to register 'charlie'...
✅ Success! Registered charlie with email charlie@example.com
```

## Hands-On Practice

```bash
cargo new custom_errors
cd custom_errors
code .
```

### **Experiment 1: Basic Custom Error**

```rust
#[derive(Debug)]
enum CalculatorError {
    DivideByZero,
    Overflow,
}

fn safe_divide(a: i32, b: i32) -> Result<i32, CalculatorError> {
    if b == 0 {
        return Err(CalculatorError::DivideByZero);
    }

    Ok(a / b)
}

fn main() {
    match safe_divide(10, 0) {
        Ok(result) => println!("Result: {}", result),
        Err(CalculatorError::DivideByZero) => println!("Cannot divide by zero"),
        Err(CalculatorError::Overflow) => println!("Overflow occurred"),
    }
}
```

### **Experiment 2: Error with Data**

```rust
use std::fmt;

#[derive(Debug)]
struct RangeError {
    value: i32,
    min: i32,
    max: i32,
}

impl fmt::Display for RangeError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(
            f,
            "Value {} out of range ({} to {})",
            self.value, self.min, self.max
        )
    }
}

fn check_range(value: i32, min: i32, max: i32) -> Result<i32, RangeError> {
    if value < min || value > max {
        Err(RangeError { value, min, max })
    } else {
        Ok(value)
    }
}

fn main() {
    match check_range(150, 0, 100) {
        Ok(v) => println!("Value {} is valid", v),
        Err(e) => println!("Error: {}", e),
    }
}
```

### **Challenge: File Operations with Custom Errors**

```rust
use std::error::Error;
use std::fmt;
use std::fs;
use std::io;

#[derive(Debug)]
enum FileOpError {
    NotFound(String),
    PermissionDenied(String),
    TooLarge { path: String, size: u64, max: u64 },
    IoError(io::Error),
}

impl fmt::Display for FileOpError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            FileOpError::NotFound(path) => write!(f, "File not found: {}", path),
            FileOpError::PermissionDenied(path) => {
                write!(f, "Permission denied: {}", path)
            }
            FileOpError::TooLarge { path, size, max } => {
                write!(
                    f,
                    "File {} too large ({} bytes, max {} bytes)",
                    path, size, max
                )
            }
            FileOpError::IoError(e) => write!(f, "IO error: {}", e),
        }
    }
}

impl Error for FileOpError {}

impl From<io::Error> for FileOpError {
    fn from(error: io::Error) -> Self {
        FileOpError::IoError(error)
    }
}

fn read_small_file(path: &str, max_size: u64) -> Result<String, FileOpError> {
    let metadata = fs::metadata(path)?;
    let size = metadata.len();

    if size > max_size {
        return Err(FileOpError::TooLarge {
            path: path.to_string(),
            size,
            max: max_size,
        });
    }

    let content = fs::read_to_string(path)?;
    Ok(content)
}

fn main() {
    // Create a test file
    fs::write("test.txt", "Hello, Rust!").expect("Failed to create test file");

    match read_small_file("test.txt", 100) {
        Ok(content) => println!("File content: {}", content),
        Err(e) => println!("Error: {}", e),
    }

    match read_small_file("test.txt", 5) {
        Ok(content) => println!("File content: {}", content),
        Err(e) => println!("Error: {}", e),  // File too large
    }

    match read_small_file("nonexistent.txt", 100) {
        Ok(content) => println!("File content: {}", content),
        Err(e) => println!("Error: {}", e),  // File not found
    }
}
```

## Key Takeaways

- ✅ Custom error enums provide specific, type-safe error handling
- ✅ Error variants can hold data for detailed error information
- ✅ Implement `Display` trait for user-friendly error messages
- ✅ Implement `Error` trait to work with `Box<dyn Error>`
- ✅ Implement `From` trait to enable automatic error conversion with `?`
- ✅ Custom errors make code self-documenting
- ✅ Pattern matching on errors allows specific error handling
- ✅ Error enums are more idiomatic than Strings
- ✅ Wrapping other error types creates application-specific errors
- ✅ Error types should be meaningful and actionable

**Next**: Practice project bringing all error handling techniques together!

---

**Progress**: Module 6, Lesson 4 complete (37/60 lessons total)
