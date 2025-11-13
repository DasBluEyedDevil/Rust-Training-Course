# Module 4, Lesson 2: Who Owns This Data? — Understanding Ownership Rules

## The Concept: One Owner at a Time

Imagine you have a physical book. At any given moment:
- **Exactly ONE person** has the book in their hands (the owner)
- The owner can read it, write in it, or lend it to someone else
- When the owner is done, they can give it away (transfer ownership)
- When no one owns it anymore, it gets thrown away

Rust enforces this same rule for data in your program. Every piece of data has **exactly one owner** at any time. This prevents multiple parts of your code from accidentally corrupting the same data.

## The Three Ownership Rules

These are THE most important rules in Rust. Memorize them:

1. **Each value in Rust has an owner**
   - Every piece of data is owned by exactly one variable

2. **There can only be one owner at a time**
   - You can't have two variables owning the same data simultaneously

3. **When the owner goes out of scope, the value is dropped**
   - When the owner variable disappears, Rust automatically cleans up the data

## Rule 1: Each Value Has an Owner

```rust
fn main() {
    let s = String::from("hello");  // s is the owner of "hello"

    println!("{}", s);
}  // s goes out of scope here, "hello" is automatically freed
```

**Visual representation:**

```
┌─────────────┐
│ Variable s  │ ───> "hello" (on heap)
│  (owner)    │
└─────────────┘
```

`s` owns the String "hello". When `s` goes away, "hello" is cleaned up.

## Rule 2: Only One Owner at a Time

**What happens when you assign one variable to another?**

### With Stack Data (Cheap Copy)

```rust
fn main() {
    let x = 5;       // x owns the value 5
    let y = x;       // y gets a COPY of 5

    println!("x: {}, y: {}", x, y);  // Both work! (5 is copied)
}
```

Stack data (integers, bools, etc.) is so cheap to copy that Rust just copies it.

### With Heap Data (Move)

```rust
fn main() {
    let s1 = String::from("hello");  // s1 owns "hello"
    let s2 = s1;                     // Ownership MOVES to s2

    // println!("{}", s1);  // ❌ ERROR! s1 no longer owns the data
    println!("{}", s2);     // ✅ OK! s2 now owns it
}
```

**What happened:**

```
BEFORE:
┌────┐
│ s1 │ ───> "hello"
└────┘

AFTER let s2 = s1:
┌────┐
│ s1 │ (invalid!)
└────┘
┌────┐
│ s2 │ ───> "hello"  (ownership transferred)
└────┘
```

Ownership **moved** from `s1` to `s2`. Now only `s2` can use the data.

**Why?** To prevent "double-free" errors. If both `s1` and `s2` tried to clean up "hello", it would cause a crash.

## Rule 3: When Owner Goes Out of Scope, Value is Dropped

```rust
fn main() {
    {
        let s = String::from("hello");  // s is valid from here

        println!("{}", s);  // use s

    }  // s goes out of scope HERE - "hello" is automatically freed

    // println!("{}", s);  // ❌ ERROR! s doesn't exist anymore
}
```

Rust automatically calls `drop()` to clean up the memory when the owner goes out of scope. You never have to manually free memory!

## Functions and Ownership

Passing a value to a function **moves ownership**:

```rust
fn main() {
    let s = String::from("hello");

    takes_ownership(s);  // s's value moves into the function

    // println!("{}", s);  // ❌ ERROR! s no longer owns the data
}

fn takes_ownership(some_string: String) {
    println!("{}", some_string);
}  // some_string goes out of scope and is dropped here
```

**Visual representation:**

```
main():
  s ───> "hello"
         ↓ (moved)
takes_ownership():
  some_string ───> "hello"
  ↓ (dropped when function ends)
  🗑️
```

## Returning Ownership from Functions

Functions can **return ownership** back:

```rust
fn main() {
    let s1 = String::from("hello");

    let s2 = takes_and_gives_back(s1);  // s1 moved in, ownership returned as s2

    // println!("{}", s1);  // ❌ ERROR! s1 was moved
    println!("{}", s2);     // ✅ OK! s2 owns it now
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string  // Return ownership to the caller
}
```

## Cloning: Making an Explicit Copy

If you want TWO variables to have the SAME data, use `.clone()`:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();  // Make a deep copy

    println!("s1: {}, s2: {}", s1, s2);  // Both work!
}
```

**Visual:**

```
┌────┐
│ s1 │ ───> "hello" (original)
└────┘
┌────┐
│ s2 │ ───> "hello" (copy)
└────┘
```

Now there are TWO separate copies in memory. Cloning is **expensive** - use it sparingly!

## Stack-Only Data: The Copy Trait

Some types are so simple that Rust automatically copies them:

```rust
fn main() {
    let x = 5;
    let y = x;  // Copy, not move!

    println!("x: {}, y: {}", x, y);  // Both work
}
```

**Types that implement `Copy` (automatic copying):**
- All integers: `i32`, `u32`, `i64`, etc.
- Booleans: `bool`
- Floating-point: `f32`, `f64`
- Characters: `char`
- Tuples of Copy types: `(i32, i32)`, `(i32, bool)`

**Types that do NOT implement `Copy` (ownership moves):**
- `String`
- `Vec<T>`
- Any type with heap-allocated data

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new ownership_rules
cd ownership_rules
code .
```

### **Experiment 1: Move Semantics**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // Move

    // Uncomment to see the error:
    // println!("{}", s1);  // ❌ ERROR!

    println!("{}", s2);  // ✅ OK
}
```

### **Experiment 2: Function Ownership**

```rust
fn main() {
    let s = String::from("world");

    print_string(s);  // s is moved into function

    // Uncomment to see the error:
    // println!("{}", s);  // ❌ ERROR! s was moved
}

fn print_string(text: String) {
    println!("{}", text);
}  // text is dropped here
```

### **Experiment 3: Returning Ownership**

```rust
fn main() {
    let s1 = gives_ownership();  // Function returns ownership
    println!("{}", s1);

    let s2 = String::from("hello");
    let s3 = takes_and_gives_back(s2);  // s2 moved, ownership returned as s3

    println!("{}", s3);
    // println!("{}", s2);  // ❌ ERROR! s2 was moved
}

fn gives_ownership() -> String {
    String::from("yours")  // Ownership given to caller
}

fn takes_and_gives_back(text: String) -> String {
    text  // Ownership returned
}
```

### **Experiment 4: Cloning vs. Moving**

```rust
fn main() {
    let s1 = String::from("hello");

    // Clone (expensive but preserves original)
    let s2 = s1.clone();
    println!("s1: {}, s2: {}", s1, s2);  // Both work

    // Move (efficient but original becomes invalid)
    let s3 = String::from("world");
    let s4 = s3;
    // println!("{}", s3);  // ❌ ERROR!
    println!("{}", s4);
}
```

### **Experiment 5: Copy Types**

```rust
fn main() {
    // Integers are Copy
    let x = 5;
    let y = x;
    println!("x: {}, y: {}", x, y);  // Both work!

    // Strings are NOT Copy (they move)
    let s1 = String::from("hello");
    let s2 = s1;
    // println!("{}", s1);  // ❌ ERROR!
    println!("{}", s2);
}
```

### **Challenge: Fix the Ownership Errors**

This code has ownership errors. Fix it by either:
- Using `.clone()`
- Returning ownership from functions
- Using different variables

```rust
fn main() {
    let s = String::from("hello");

    calculate_length(s);
    println!("{}", s);  // ❌ ERROR! Fix this
}

fn calculate_length(text: String) {
    println!("Length: {}", text.len());
}
```

**Solution 1: Clone**
```rust
calculate_length(s.clone());
```

**Solution 2: Return ownership**
```rust
fn calculate_length(text: String) -> String {
    println!("Length: {}", text.len());
    text  // Return ownership
}

let s = calculate_length(s);
```

**Solution 3: Use references (next lesson!)**
```rust
// We'll learn this in Lesson 4!
```

## Key Takeaways

- ✅ **Rule 1**: Each value has exactly one owner
- ✅ **Rule 2**: Only one owner at a time (moves, not copies for heap data)
- ✅ **Rule 3**: When owner goes out of scope, value is dropped (automatic cleanup)
- ✅ Passing to functions **moves** ownership
- ✅ Returning from functions **gives** ownership back
- ✅ `.clone()` makes an explicit deep copy
- ✅ Stack data (i32, bool, etc.) implements `Copy` - automatically copied
- ✅ Heap data (String, Vec) is **moved** by default

## Why This Matters

Ownership prevents:
- **Memory leaks** (forgetting to free memory)
- **Double-free errors** (freeing the same memory twice)
- **Use-after-free errors** (using memory after it's freed)
- **Data races** (multiple parts of code modifying the same data)

All of this happens **at compile time** with zero runtime cost!

**Next lesson**: We'll learn about **borrowing** - a way to let functions use data WITHOUT taking ownership!
