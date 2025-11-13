# Module 1, Lesson 2: Running Your Code with `cargo run`

## The Translation Process

Imagine you write a recipe in English, but you're giving it to a robot chef that only understands a special robot language. You need a translator to convert your English recipe into robot instructions before the robot can make cookies.

Programming is the same:
- **You write**: Rust code (human-readable)
- **Computer needs**: Machine code (binary—ones and zeros)
- **The solution**: A translator called a **compiler**

The Rust compiler (called `rustc`) translates your `.rs` files into executable programs.

## What is Compilation?

**Compilation** is the process of translating your code from human-readable Rust into machine code the computer can execute.

```
Your Code (main.rs)  →  [Compiler/rustc]  →  Executable Binary
  Human-readable          Translation          Machine code
```

The **executable** (also called a **binary**) is the actual program you run.

## The `cargo run` Command

Instead of manually compiling and running, you use **`cargo run`**, which:
1. Checks if your code changed since the last compilation
2. If yes: Compiles your code
3. Runs the resulting executable
4. Shows any output

If nothing changed, it skips compilation and just runs the existing executable.

## What You See When Running cargo run

Given this code:

```rust
fn main() {
    println!("Hello from Rust!");
}
```

When you run `cargo run`:

```
   Compiling hello v0.1.0 (/path/to/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/hello`
Hello from Rust!
```

**Breaking it down:**

1. **Compiling hello v0.1.0** — Cargo is translating your code
2. **Finished dev** — Compilation successful (debug mode)
3. **Running target/debug/hello** — Cargo is executing the program
4. **Hello from Rust!** — Your program's output

**Run again without changes:**

```
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/hello`
Hello from Rust!
```

No "Compiling" line—Cargo is smart and skipped recompilation!

## Hands-On Practice

### **Create a Test Project**

```bash
cargo new run_practice
cd run_practice
```

### **Experiment 1: Run Twice**

```bash
cargo run   # First run - will compile
cargo run   # Second run - skips compilation
```

Notice the difference in output!

### **Experiment 2: Modify and Rerun**

1. Change the message in `src/main.rs`
2. Save the file
3. Run `cargo run`

The "Compiling" line appears again—Cargo detected your change.

### **Experiment 3: Find the Executable**

After `cargo run`, check `target/debug/`:

```bash
ls target/debug/        # macOS/Linux
dir target\debug\       # Windows
```

You'll see `run_practice` (or `run_practice.exe` on Windows). This is your compiled program!

Run it directly:

```bash
./target/debug/run_practice        # macOS/Linux
.\target\debug\run_practice.exe    # Windows
```

Same output as `cargo run`!

### **Experiment 4: cargo build vs. cargo run**

Try this:

```bash
cargo build   # Only compiles, doesn't run
```

You'll see "Compiling" and "Finished," but no output from your program. To run it, you'd execute the binary manually.

**When to use each:**
- **`cargo run`** — Compile and run (use this 99% of the time)
- **`cargo build`** — Only compile (when you want the executable but don't need to run it yet)

### **Experiment 5: Compilation Errors**

Introduce an error in `main.rs`:

```rust
fn main() {
    println!("Hello")  // ❌ Missing semicolon
}
```

Run `cargo run`:

```
error: expected `;`, found `}`
```

The compiler caught the mistake! Your program never ran because it failed to compile.

Fix it and run again—now it works.

## Key Takeaways

- ✅ **Compilation** translates Rust code into machine code (executable)
- ✅ **`cargo run`** compiles (if needed) and runs your program
- ✅ Cargo is smart—it only recompiles when code changes
- ✅ The compiled executable lives in `target/debug/`
- ✅ You can run the executable directly without Cargo
- ✅ **`cargo build`** compiles without running
- ✅ Compilation errors prevent your program from running
