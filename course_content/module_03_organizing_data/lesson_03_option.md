# Module 3, Lesson 3: When Something Might Not Exist — The `Option` Enum

## The Concept: Maybe There, Maybe Not

Imagine looking in your refrigerator for milk. There are two possibilities:
- **There IS milk** — You have Some milk
- **There is NO milk** — You have None

Many programming languages represent "no value" with `null`, but this causes billions of dollars in bugs every year. Rust takes a different approach: if a value might not exist, you must explicitly handle both cases using the `Option` enum.

## The `Option` Enum

`Option` is a built-in Rust enum with two variants:

```rust
enum Option<T> {
    Some(T),    // There IS a value of type T
    None,       // There is NO value
}
```

**Don't worry about `<T>` yet**—just know it means Option can hold any type.

**Examples:**

```rust
let some_number: Option<i32> = Some(5);
let no_number: Option<i32> = None;

let some_string: Option<String> = Some(String::from("hello"));
let no_string: Option<String> = None;
```

## Why Use Option?

**The problem with null in other languages:**

```java
// Java example (dangerous!)
String name = getName();  // Might return null
int length = name.length();  // CRASH if name is null!
```

**The Rust way (safe):**

```rust
fn get_name() -> Option<String> {
    // Might return Some(name) or None
}

let name_option = get_name();

match name_option {
    Some(name) => println!("Length: {}", name.len()),
    None => println!("No name provided"),
}
```

The compiler **forces** you to handle the `None` case!

## Using Option with match

```rust
fn find_word_at_index(index: usize) -> Option<&'static str> {
    let words = ["apple", "banana", "cherry"];

    if index < words.len() {
        Some(words[index])
    } else {
        None
    }
}

fn main() {
    let result = find_word_at_index(1);

    match result {
        Some(word) => println!("Found: {}", word),
        None => println!("Index out of bounds"),
    }
}
```

Output: `Found: banana`

## Useful Option Methods

**Check if there's a value:**

```rust
let x: Option<i32> = Some(5);

if x.is_some() {
    println!("Has a value!");
}

if x.is_none() {
    println!("No value!");
}
```

**Get the value (or panic):**

```rust
let x: Option<i32> = Some(5);
let value = x.unwrap();  // Gets 5, but panics if None!
println!("{}", value);

// Safer: provide a default
let y: Option<i32> = None;
let value = y.unwrap_or(0);  // Returns 0 if None
println!("{}", value);
```

**Transform the value:**

```rust
let x: Option<i32> = Some(5);

let doubled = x.map(|n| n * 2);  // Some(10)
println!("{:?}", doubled);

let nothing: Option<i32> = None;
let doubled_nothing = nothing.map(|n| n * 2);  // Still None
println!("{:?}", doubled_nothing);
```

## if let Syntax

For handling only the `Some` case:

```rust
fn main() {
    let some_number = Some(7);

    // Instead of match:
    match some_number {
        Some(n) => println!("Number: {}", n),
        None => {},
    }

    // Use if let (cleaner):
    if let Some(n) = some_number {
        println!("Number: {}", n);
    }
}
```

## Practical Examples

**Finding a value in a list:**

```rust
fn find_even(numbers: Vec<i32>) -> Option<i32> {
    for num in numbers {
        if num % 2 == 0 {
            return Some(num);  // Found one!
        }
    }
    None  // Didn't find any
}

fn main() {
    let nums = vec![1, 3, 5, 8, 9];

    match find_even(nums) {
        Some(n) => println!("First even number: {}", n),
        None => println!("No even numbers found"),
    }
}
```

**User input that might be empty:**

```rust
fn get_user_age() -> Option<i32> {
    // Simulating user input
    let input = "";  // Empty input

    if input.is_empty() {
        None
    } else {
        Some(input.parse().unwrap_or(0))
    }
}

fn main() {
    match get_user_age() {
        Some(age) => println!("User is {} years old", age),
        None => println!("No age provided"),
    }
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new option_practice
cd option_practice
code .
```

### **Experiment 1: Basic Option**

```rust
fn divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None  // Can't divide by zero
    } else {
        Some(a / b)
    }
}

fn main() {
    let result1 = divide(10.0, 2.0);
    let result2 = divide(10.0, 0.0);

    match result1 {
        Some(value) => println!("Result: {}", value),
        None => println!("Cannot divide by zero!"),
    }

    match result2 {
        Some(value) => println!("Result: {}", value),
        None => println!("Cannot divide by zero!"),
    }
}
```

### **Experiment 2: Finding Values**

```rust
fn find_first_uppercase(text: &str) -> Option<char> {
    for c in text.chars() {
        if c.is_uppercase() {
            return Some(c);
        }
    }
    None
}

fn main() {
    let text1 = "hello World";
    let text2 = "no uppercase here";

    if let Some(ch) = find_first_uppercase(text1) {
        println!("First uppercase: {}", ch);
    } else {
        println!("No uppercase letters");
    }

    if let Some(ch) = find_first_uppercase(text2) {
        println!("First uppercase: {}", ch);
    } else {
        println!("No uppercase letters");
    }
}
```

### **Challenge: Safe Array Access**

Create a function that safely gets an element from an array:

```rust
fn get_element(arr: &[i32], index: usize) -> Option<i32> {
    if index < arr.len() {
        Some(arr[index])
    } else {
        None
    }
}

fn main() {
    let numbers = [10, 20, 30, 40, 50];

    // Try different indices
    for i in 0..7 {
        match get_element(&numbers, i) {
            Some(value) => println!("Index {}: {}", i, value),
            None => println!("Index {} is out of bounds", i),
        }
    }
}
```

## Key Takeaways

- ✅ `Option<T>` represents a value that might not exist
- ✅ `Some(value)` means there IS a value
- ✅ `None` means there is NO value
- ✅ Replaces null from other languages
- ✅ Compiler forces you to handle both cases
- ✅ Use `match`, `if let`, or methods like `.unwrap_or()`
- ✅ Makes code safer by preventing null pointer errors
- ✅ `.is_some()`, `.is_none()`, `.map()` are useful methods
