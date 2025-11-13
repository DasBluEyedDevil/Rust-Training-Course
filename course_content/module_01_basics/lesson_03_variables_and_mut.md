# Module 1, Lesson 3: Variables and Boxes — Storing Information (`let`, `mut`)

## The Concept: Labeled Boxes

Imagine organizing your desk with sticky notes to remember important information:
- Friend's phone number: "555-1234"
- Book to read: "The Hobbit"
- Apples you have: 5

Each sticky note has:
- A **label** (what it represents)
- A **value** (the actual information)

Now imagine two types of sticky notes:

1. **Permanent marker** — Once written, can't be changed
2. **Pencil** — Can be erased and rewritten

In Rust, you store information in "labeled boxes" called **variables**. By default, Rust gives you permanent marker boxes—once you set a value, it can't change. If you want to change a value later, you explicitly ask for a pencil box.

This prevents bugs caused by accidentally changing values you didn't mean to change.

## Creating Variables: The `let` Keyword

To create a variable, use **`let`**:

```rust
let apple_count = 5;
```

This means: "Create a variable named `apple_count` and store the value `5` in it."

**Important: Variables are immutable by default**

**Immutable** means "cannot be changed":

```rust
let score = 100;
score = 150;  // ❌ ERROR: cannot assign twice to immutable variable
```

Once you create a variable with `let`, you can't modify its value.

## Making Variables Changeable: The `mut` Keyword

If you want a variable that CAN be modified, use **`mut`** (short for "mutable"):

```rust
let mut score = 100;  // Now it's mutable
score = 150;          // ✅ OK: we can change it
```

Think of it this way:
- **`let`** = permanent marker (immutable)
- **`let mut`** = pencil (mutable)

## Why Immutable by Default?

Rust's philosophy: "If something doesn't need to change, it shouldn't be allowed to change."

This prevents bugs. Imagine:

```rust
let max_lives = 3;  // Game rule: 3 lives

// ... 100 lines of code ...

max_lives = 5;  // ❌ Compiler error!
```

If this were allowed, you might accidentally break your game logic. Immutability protects you.

## Code Examples

**Creating multiple variables:**

```rust
fn main() {
    let name = "Alice";
    let age = 30;
    let is_student = true;

    println!("Name: {}", name);
    println!("Age: {}", age);
    println!("Is student: {}", is_student);
}
```

Output:
```
Name: Alice
Age: 30
Is student: true
```

**Trying to change an immutable variable (ERROR):**

```rust
fn main() {
    let score = 100;
    score = 150;  // ❌ ERROR
}
```

Error:
```
error[E0384]: cannot assign twice to immutable variable `score`
```

**Using mut to allow changes:**

```rust
fn main() {
    let mut score = 100;
    println!("Initial: {}", score);

    score = 150;
    println!("Updated: {}", score);

    score = score + 50;
    println!("Final: {}", score);
}
```

Output:
```
Initial: 100
Updated: 150
Final: 200
```

## Hands-On Practice

### **Create a Test Project**

```bash
cargo new variables_practice
cd variables_practice
code .
```

### **Experiment 1: Immutable Variables**

```rust
fn main() {
    let name = "Bob";
    let favorite_number = 42;

    println!("My name is {}", name);
    println!("My favorite number is {}", favorite_number);
}
```

Run `cargo run`.

### **Experiment 2: Try to Change an Immutable Variable**

Add this line:

```rust
favorite_number = 100;  // Try to change it
```

Run `cargo run`. You'll get an error. Read it carefully!

Remove that line before continuing.

### **Experiment 3: Mutable Variables**

```rust
fn main() {
    let mut counter = 0;
    println!("Counter: {}", counter);

    counter = counter + 1;
    println!("Counter: {}", counter);

    counter = counter + 1;
    println!("Counter: {}", counter);
}
```

Output:
```
Counter: 0
Counter: 1
Counter: 2
```

### **Experiment 4: Temperature Tracker**

```rust
fn main() {
    let mut temperature = 20;

    println!("Morning: {}°C", temperature);

    temperature = temperature + 5;
    println!("Afternoon: {}°C", temperature);

    temperature = temperature - 3;
    println!("Evening: {}°C", temperature);
}
```

### **Challenge: Score Tracker**

Write a program that:
1. Starts with `score = 0`
2. Adds 10 points, prints it
3. Adds 25 points, prints it
4. Subtracts 5 points, prints it

Expected output: `0 → 10 → 35 → 30`

## Key Takeaways

- ✅ **Variables** store labeled values
- ✅ **`let`** creates an immutable (unchangeable) variable
- ✅ **`let mut`** creates a mutable (changeable) variable
- ✅ Immutability by default prevents accidental bugs
- ✅ Use `mut` only when you need to change a value
- ✅ `println!("Value: {}", var)` displays variable values
