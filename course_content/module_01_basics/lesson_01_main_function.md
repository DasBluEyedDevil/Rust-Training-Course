# Module 1, Lesson 1: The `main()` Function — Where Your Program Starts

## The Entry Point Concept

Imagine reading a book. You start at page 1, chapter 1—not at some random page in the middle. The same is true for computer programs. When you run a program, the computer needs to know where to start.

In Rust, that starting point is always the same: a special function called **main**.

When you run a Rust program:
1. The computer looks for a function named `main`
2. It starts executing code from the first line inside `main()`
3. It runs each line in order, from top to bottom
4. When it reaches the end of `main()`, the program exits

Think of `main()` as the front door to your program.

## The main() Function Syntax

```rust
fn main() {
    // Your code goes here
}
```

**Breaking it down:**
- `fn` — Keyword meaning "function"
- `main` — The name (must be exactly "main" for the entry point)
- `()` — Parentheses (parameters go here, empty for now)
- `{ }` — Curly braces containing the code to execute

## Code Examples

**The simplest valid Rust program:**
```rust
fn main() {
    // Does nothing, but it compiles and runs!
}
```

**A program that produces output:**
```rust
fn main() {
    println!("Line 1");
    println!("Line 2");
    println!("Line 3");
}
```

Output:
```
Line 1
Line 2
Line 3
```

The code executes from top to bottom, one line at a time.

**What happens without main():**
```rust
// ❌ This will NOT compile
println!("Hello!");
```

Error:
```
error[E0601]: `main` function not found
```

Rust requires a `main()` function—it's the only way the compiler knows where your program begins.

## Hands-On Practice

### **Create a Test Project**

```bash
cargo new main_practice
cd main_practice
code .
```

### **Experiment 1: Add Multiple Lines**

Modify `src/main.rs`:

```rust
fn main() {
    println!("Starting program");
    println!("Doing something");
    println!("Doing something else");
    println!("Ending program");
}
```

Run `cargo run` and observe the output order.

### **Experiment 2: Remove main() and See the Error**

Delete everything and write:

```rust
println!("Hello!");
```

Run `cargo run`. You'll get: `error: main function not found`

This error teaches you that `main()` is mandatory.

Add it back:

```rust
fn main() {
    println!("Hello!");
}
```

Now it works!

### **Experiment 3: Capitalization Matters**

Try this:

```rust
fn Main() {  // ❌ Wrong - capital M
    println!("Hello!");
}
```

Run `cargo run`. Error: `main function not found`

Rust is case-sensitive. It must be lowercase: `main`

### **Experiment 4: Prove Execution Order**

```rust
fn main() {
    println!("First");
    println!("Second");
    println!("Third");
}
```

Run it. Now rearrange the lines:

```rust
fn main() {
    println!("Third");
    println!("First");
    println!("Second");
}
```

Run it again—the output changes. This proves code runs from top to bottom.

## Key Takeaways

- ✅ Every Rust program needs exactly one `main()` function
- ✅ `main()` is the entry point—where execution begins
- ✅ Code inside `main()` runs from top to bottom
- ✅ The function must be named `main` (lowercase)
- ✅ The syntax is: `fn main() { ... }`
- ✅ If there's no `main()`, the program won't compile
