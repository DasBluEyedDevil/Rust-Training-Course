# Module 1, Lesson 5: Functions — Teaching Your Program New Tricks

## The Concept: Reusable Recipes

Imagine you teach a friend how to make coffee:
1. Grind the beans
2. Boil water
3. Brew for 4 minutes
4. Pour into cup

Now, instead of repeating these steps every time, you just say **"make coffee"** and your friend knows exactly what to do.

Functions work the same way in programming. Instead of writing the same code over and over, you:
1. **Define** a function once (teach the recipe)
2. **Call** the function whenever you need it (say "do the thing")

## Creating Functions

A **function** is a named block of code that performs a specific task.

**Syntax:**

```rust
fn function_name() {
    // Code goes here
}
```

- `fn` — Keyword for "function"
- `function_name` — The name you give it (use snake_case: lowercase with underscores)
- `()` — Parameters go here (empty for now)
- `{ }` — The code to execute

**Example:**

```rust
fn greet() {
    println!("Hello!");
    println!("Welcome to Rust!");
}

fn main() {
    greet();  // Call the function
    greet();  // Call it again
}
```

Output:
```
Hello!
Welcome to Rust!
Hello!
Welcome to Rust!
```

The `greet()` code runs twice, but we only wrote it once!

## Functions with Parameters

**Parameters** let you pass information into a function.

Think of it like: "Make coffee for 2 people" vs. "Make coffee for 5 people"—the number of people is a parameter.

**Syntax:**

```rust
fn function_name(parameter_name: Type) {
    // Use parameter_name here
}
```

**Example:**

```rust
fn greet_person(name: &str) {
    println!("Hello, {}!", name);
    println!("Nice to meet you!");
}

fn main() {
    greet_person("Alice");
    greet_person("Bob");
    greet_person("Charlie");
}
```

Output:
```
Hello, Alice!
Nice to meet you!
Hello, Bob!
Nice to meet you!
Hello, Charlie!
Nice to meet you!
```

**Multiple parameters:**

```rust
fn introduce(name: &str, age: i32) {
    println!("My name is {} and I'm {} years old", name, age);
}

fn main() {
    introduce("Alice", 30);
    introduce("Bob", 25);
}
```

## Functions that Return Values

Functions can **return** a result back to the caller.

Think of it like asking: "What's 5 + 3?" and getting back "8".

**Syntax:**

```rust
fn function_name() -> ReturnType {
    // Code here
    return_value  // No semicolon on the last expression!
}
```

**Example:**

```rust
fn add(x: i32, y: i32) -> i32 {
    x + y  // Return the sum (no semicolon!)
}

fn main() {
    let result = add(5, 3);
    println!("5 + 3 = {}", result);

    let another_result = add(10, 20);
    println!("10 + 20 = {}", another_result);
}
```

Output:
```
5 + 3 = 8
10 + 20 = 30
```

**Important:** The last expression in a function (without a semicolon) is automatically returned.

You can also use `return` explicitly:

```rust
fn add(x: i32, y: i32) -> i32 {
    return x + y;  // Explicit return with semicolon
}
```

Both styles work, but omitting `return` is more idiomatic in Rust.

## Complete Example

```rust
fn main() {
    greet();

    let temp_f = celsius_to_fahrenheit(25.0);
    println!("25°C is {}°F", temp_f);

    let area = rectangle_area(5, 10);
    println!("Rectangle area: {}", area);
}

fn greet() {
    println!("Welcome to the program!");
}

fn celsius_to_fahrenheit(celsius: f64) -> f64 {
    celsius * 1.8 + 32.0
}

fn rectangle_area(width: i32, height: i32) -> i32 {
    width * height
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new functions_practice
cd functions_practice
code .
```

### **Experiment 1: Simple Function**

```rust
fn say_hello() {
    println!("Hello, world!");
}

fn main() {
    say_hello();
    say_hello();
    say_hello();
}
```

### **Experiment 2: Function with Parameters**

```rust
fn print_number(n: i32) {
    println!("The number is: {}", n);
}

fn main() {
    print_number(5);
    print_number(100);
    print_number(-42);
}
```

### **Experiment 3: Function that Returns a Value**

```rust
fn square(n: i32) -> i32 {
    n * n
}

fn main() {
    let result = square(5);
    println!("5 squared is {}", result);

    println!("10 squared is {}", square(10));
}
```

### **Challenge: Create a Calculator**

Write four functions:
- `add(a, b)` — Returns `a + b`
- `subtract(a, b)` — Returns `a - b`
- `multiply(a, b)` — Returns `a * b`
- `divide(a, b)` — Returns `a / b`

Then use them in `main()`:

```rust
fn main() {
    println!("10 + 5 = {}", add(10, 5));
    println!("10 - 5 = {}", subtract(10, 5));
    println!("10 * 5 = {}", multiply(10, 5));
    println!("10 / 5 = {}", divide(10, 5));
}

fn add(a: i32, b: i32) -> i32 {
    a + b
}

// Write subtract, multiply, and divide here!
```

## Key Takeaways

- ✅ Functions are reusable blocks of code
- ✅ `fn name() { }` defines a function
- ✅ Call a function with `name()`
- ✅ Parameters pass data into functions: `fn name(param: Type)`
- ✅ Return values with `-> Type`
- ✅ Last expression (without `;`) is automatically returned
- ✅ Functions make code more organized and reusable
