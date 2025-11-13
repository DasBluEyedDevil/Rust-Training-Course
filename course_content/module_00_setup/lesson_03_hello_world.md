# Module 0, Lesson 3: Your First "Hello, World!" — Making the Computer Say Hello

## Why "Hello, World"?

When learning any new programming language, the tradition is to write a simple program that displays "Hello, World!" on the screen. It's like learning to say "Hello!" when studying a new human language.

This simple program proves that:
- Your tools are installed correctly
- You can write code that the computer understands
- You can run that code and see results
- Everything is working!

## How Programs Work: The Basic Cycle

Every program you write goes through three steps:

1. **Write**: You type code in a file
2. **Compile**: A tool translates your code into instructions the computer understands
3. **Run**: The computer executes those instructions

In Rust, we use **Cargo** to do all three steps with one command: `cargo run`

## The Anatomy of a Rust Program

Here's a complete Rust program:

```rust
fn main() {
    println!("Hello, World!");
}
```

**Breaking it down:**

- `fn main()` — This is the "main function," the entry point of your program. Every Rust program starts here.
- `{ }` — Curly braces contain the code that runs
- `println!(...)` — A command that prints text to the screen (the `!` makes it a "macro"—more on that later)
- `"Hello, World!"` — The text to display (must be in double quotes)
- `;` — Semicolon ends the statement (like a period ends a sentence)

## Create Your First Program

### **Step 1: Create a New Project**

Open your terminal and run:

```bash
cargo new hello
cd hello
```

This creates a new folder called `hello` with your project inside.

### **Step 2: Open It in VS Code**

```bash
code .
```

(The `.` means "open the current folder")

In VS Code, open the file `src/main.rs`. You'll see Cargo already created a "Hello, World!" program for you!

### **Step 3: Run It**

In your terminal (inside the `hello` folder), run:

```bash
cargo run
```

**You should see:**

```
   Compiling hello v0.1.0 (/path/to/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.23s
     Running `target/debug/hello`
Hello, world!
```

The last line is your program's output. **You did it!**

### **Step 4: Make It Personal**

Change the text in `src/main.rs`:

```rust
fn main() {
    println!("Hello, my name is Alex!");
    println!("I'm learning Rust!");
}
```

Save the file and run `cargo run` again. You'll see your custom messages!

## Experiments: Learn by Breaking Things

The best way to learn is to experiment. Try these:

**Experiment 1: Remove the semicolon**
```rust
fn main() {
    println!("Hello!")  // ← No semicolon
}
```
Run `cargo run`. What error do you get? Read it carefully, then add the semicolon back.

**Experiment 2: Remove the exclamation mark**
```rust
fn main() {
    println("Hello!");  // ← No exclamation mark
}
```
Run `cargo run`. What does the error say? Add the `!` back.

**Experiment 3: Print multiple lines**
```rust
fn main() {
    println!("Line 1");
    println!("Line 2");
    println!("Line 3");
}
```

## What You've Learned

- ✅ `cargo new` creates a new Rust project
- ✅ `cargo run` compiles and runs your program
- ✅ `fn main()` is where every program starts
- ✅ `println!()` prints text to the screen
- ✅ Semicolons are required at the end of statements
- ✅ Compiler errors are helpful—they tell you exactly what's wrong!
