# Module 1, Lesson 6: Comments — Leaving Notes for Yourself

## The Concept: Notes in Your Code

Imagine you're cooking with a recipe book. Some recipes have helpful notes like:
- "This step takes 30 minutes"
- "You can substitute butter for oil"
- "Be careful not to overheat!"

These notes aren't ingredients or instructions—they're explanations to help you understand what's happening.

In programming, **comments** are notes you leave in your code to:
- Explain what the code does
- Remind yourself why you did something
- Help others understand your code
- Temporarily disable code without deleting it

The computer completely ignores comments—they're only for humans.

## Types of Comments in Rust

### **1. Single-Line Comments (`//`)**

Everything after `//` on that line is a comment:

```rust
fn main() {
    // This is a comment
    println!("Hello!");  // This is also a comment

    // Comments can explain what the next line does
    let age = 25;
}
```

### **2. Multi-Line Comments (`/* */`)**

For longer explanations:

```rust
fn main() {
    /*
       This is a multi-line comment.
       It can span multiple lines.
       Useful for longer explanations.
    */
    println!("Hello!");

    /* You can also use it on one line */
    let x = 5;
}
```

### **3. Documentation Comments (`///`)**

Special comments that generate documentation (we'll use these later):

```rust
/// This function greets a person by name.
///
/// # Examples
/// ```
/// greet_person("Alice");
/// ```
fn greet_person(name: &str) {
    println!("Hello, {}!", name);
}
```

For now, stick with `//` comments—they're the most common.

## When to Use Comments

**Good uses:**

```rust
fn main() {
    // Convert Celsius to Fahrenheit
    let celsius = 25.0;
    let fahrenheit = celsius * 1.8 + 32.0;

    // Display the result
    println!("{}°C is {}°F", celsius, fahrenheit);
}
```

**Explaining WHY, not WHAT:**

```rust
// Good: Explains WHY
// We use i64 here because the number can exceed i32's limit
let world_population: i64 = 7_900_000_000;

// Bad: Just repeats what the code says
// Create a variable called x and set it to 5
let x = 5;
```

**Good comments explain the purpose, not the syntax.**

## Commenting Out Code

Use comments to temporarily disable code:

```rust
fn main() {
    println!("This runs");
    // println!("This doesn't run");
    println!("This runs too");
}
```

Output:
```
This runs
This runs too
```

Useful while debugging or testing!

## Complete Example

```rust
fn main() {
    // Temperature conversion program
    // Takes a Celsius temperature and converts it to Fahrenheit

    let celsius = 25.0;  // Starting temperature

    // Formula: F = C * 1.8 + 32
    let fahrenheit = celsius * 1.8 + 32.0;

    // Display the result
    println!("{}°C equals {}°F", celsius, fahrenheit);

    // Uncomment the line below to test with a different temperature
    // celsius_to_fahrenheit(100.0);
}

/// Converts Celsius to Fahrenheit and prints the result
fn celsius_to_fahrenheit(c: f64) {
    let f = c * 1.8 + 32.0;
    println!("{}°C equals {}°F", c, f);
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new comments_practice
cd comments_practice
code .
```

### **Experiment 1: Add Comments**

```rust
fn main() {
    // This program calculates the area of a rectangle

    let width = 10;   // Width in meters
    let height = 5;   // Height in meters

    // Calculate area (width * height)
    let area = width * height;

    println!("The area is {} square meters", area);
}
```

### **Experiment 2: Comment Out Code**

```rust
fn main() {
    println!("Line 1");
    // println!("Line 2 is commented out");
    println!("Line 3");

    /*
    println!("Line 4 is in a multi-line comment");
    println!("Line 5 too");
    */

    println!("Line 6");
}
```

Run it and see which lines print!

### **Experiment 3: Multi-Line Comments**

```rust
fn main() {
    /*
       This is a calculator program.
       It performs basic arithmetic operations.
       Created for learning purposes.
    */

    let x = 10;
    let y = 3;

    println!("Sum: {}", x + y);
    println!("Difference: {}", x - y);
}
```

### **Challenge: Document Your Code**

Take the temperature converter from Lesson 4 and add helpful comments:

```rust
fn main() {
    // TODO: Add comments explaining what this program does

    let celsius = 25.0;
    let fahrenheit = celsius * 1.8 + 32.0;

    println!("{}°C is {}°F", celsius, fahrenheit);
}
```

Add comments that explain:
1. What the program does
2. What the conversion formula means
3. What the variables represent

## Key Takeaways

- ✅ Comments are notes for humans, ignored by the compiler
- ✅ `//` creates a single-line comment
- ✅ `/* */` creates a multi-line comment
- ✅ Use comments to explain WHY, not WHAT
- ✅ Comments can temporarily disable code
- ✅ Good comments make code easier to understand
- ✅ Don't over-comment obvious things

---

## ✅ Module 1 Complete!

You've learned the fundamentals of Rust:
- ✅ The `main()` function (entry point)
- ✅ Running code with `cargo run`
- ✅ Variables (`let`, `mut`)
- ✅ Basic data types (i32, f64, bool, char, &str)
- ✅ Functions (defining and calling)
- ✅ Comments (documenting your code)

**You're ready to start making decisions and controlling program flow!**
