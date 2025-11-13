# Module 2, Lesson 5: Practice Project — Building a Simple Calculator

## Project Overview

Now that you've learned `if/else`, `loop`, `while`, and `for`, let's combine everything into a real project: a simple command-line calculator!

**What we're building:**
- A program that performs basic math operations
- User chooses the operation (add, subtract, multiply, divide)
- Calculator runs in a loop until the user quits
- Handles division by zero errors

**Skills practiced:**
- ✅ if/else for decision-making
- ✅ loop for the main program loop
- ✅ Functions to organize code
- ✅ Variables and basic types
- ✅ User input (bonus: we'll learn this now!)

## Getting User Input

Before building the calculator, let's learn how to get input from the user:

```rust
use std::io;  // Import input/output library

fn main() {
    println!("Enter your name:");

    let mut input = String::new();  // Create an empty string

    io::stdin()
        .read_line(&mut input)  // Read user input
        .expect("Failed to read line");  // Handle errors

    println!("Hello, {}!", input.trim());  // trim() removes whitespace
}
```

**Breaking it down:**
- `use std::io` — Import Rust's input/output library
- `String::new()` — Create a new, empty String
- `io::stdin().read_line(&mut input)` — Read a line from the terminal
- `.trim()` — Remove newline characters

## Parsing Numbers from Input

To convert user input to a number:

```rust
use std::io;

fn main() {
    println!("Enter a number:");

    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to read");

    let number: i32 = input.trim().parse().expect("Please enter a number!");

    println!("You entered: {}", number);
}
```

## The Calculator Project

### **Step 1: Create the Project**

```bash
cargo new calculator
cd calculator
code .
```

### **Step 2: Build the Calculator (Complete Code)**

Replace `src/main.rs` with:

```rust
use std::io;

fn main() {
    println!("=== Simple Calculator ===");
    println!("Type 'quit' to exit\n");

    loop {
        // Get first number
        println!("Enter first number:");
        let num1 = get_number();

        // Get operation
        println!("Enter operation (+, -, *, /):");
        let operation = get_input();

        // Check if user wants to quit
        if operation.trim() == "quit" {
            println!("Goodbye!");
            break;
        }

        // Get second number
        println!("Enter second number:");
        let num2 = get_number();

        // Perform calculation
        let result = calculate(num1, num2, operation.trim());

        // Display result
        match result {
            Some(value) => println!("\nResult: {}\n", value),
            None => println!("\nError: Invalid operation or division by zero\n"),
        }
    }
}

// Function to get user input as a string
fn get_input() -> String {
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read input");
    input
}

// Function to get a number from the user
fn get_number() -> f64 {
    loop {
        let input = get_input();

        match input.trim().parse() {
            Ok(num) => return num,
            Err(_) => println!("Invalid number, try again:"),
        }
    }
}

// Function to perform the calculation
fn calculate(num1: f64, num2: f64, operation: &str) -> Option<f64> {
    match operation {
        "+" => Some(num1 + num2),
        "-" => Some(num1 - num2),
        "*" => Some(num1 * num2),
        "/" => {
            if num2 == 0.0 {
                None  // Can't divide by zero
            } else {
                Some(num1 / num2)
            }
        }
        _ => None,  // Invalid operation
    }
}
```

### **Step 3: Run the Calculator**

```bash
cargo run
```

**Try it out:**
```
=== Simple Calculator ===
Type 'quit' to exit

Enter first number:
10
Enter operation (+, -, *, /):
+
Enter second number:
5

Result: 15

Enter first number:
20
Enter operation (+, -, *, /):
/
Enter second number:
4

Result: 5

Enter first number:
quit
Goodbye!
```

## Understanding the Code

**Main loop:**
```rust
loop {
    // Get inputs
    // Perform calculation
    // Display result
    // Repeat until user quits
}
```

**Getting input:**
```rust
fn get_number() -> f64 {
    loop {
        // Try to parse input
        // If successful, return the number
        // If failed, ask again
    }
}
```

**Calculation logic:**
```rust
match operation {
    "+" => Some(num1 + num2),
    // ... other operations
    _ => None,  // Catch-all for invalid operations
}
```

We use `Option` (Some/None) to handle errors like division by zero (we'll learn more about `Option` in Module 3!).

## Challenges and Extensions

### **Challenge 1: Add More Operations**

Add these operations:
- `%` — Modulo (remainder)
- `^` — Power (hint: use `num1.powf(num2)`)

### **Challenge 2: Keep Running Total**

Modify the calculator to keep a running total:
```
Result: 10
Do you want to continue with 10? (yes/no):
yes
Enter operation:
+
Enter number:
5
Result: 15
```

### **Challenge 3: Better Error Messages**

Add specific error messages:
- "Cannot divide by zero!"
- "Unknown operation!"
- "Invalid number format!"

## Simplified Version

If the full version is too complex, try this simpler version first:

```rust
fn main() {
    println!("Enter first number:");
    let mut input = String::new();
    std::io::stdin().read_line(&mut input).unwrap();
    let num1: f64 = input.trim().parse().unwrap();

    println!("Enter second number:");
    input.clear();
    std::io::stdin().read_line(&mut input).unwrap();
    let num2: f64 = input.trim().parse().unwrap();

    println!("\n{} + {} = {}", num1, num2, num1 + num2);
    println!("{} - {} = {}", num1, num2, num1 - num2);
    println!("{} * {} = {}", num1, num2, num1 * num2);
    println!("{} / {} = {}", num1, num2, num1 / num2);
}
```

## Key Takeaways

- ✅ Combined `loop`, `if/else`, and functions
- ✅ Learned basic user input with `std::io`
- ✅ Converted strings to numbers with `.parse()`
- ✅ Handled errors (division by zero, invalid input)
- ✅ Created a real, usable program!
- ✅ Organized code with multiple functions

---

## ✅ Module 2 Complete!

You've mastered program flow:
- ✅ if/else (making decisions)
- ✅ loop (infinite loops with break)
- ✅ while (conditional loops)
- ✅ for (iterating over collections)
- ✅ Built a complete calculator project!

**Next: Module 3 — Organizing your data with structs and enums!**
