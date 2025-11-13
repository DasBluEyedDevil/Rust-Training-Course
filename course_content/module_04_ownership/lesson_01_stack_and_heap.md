# Module 4, Lesson 1: Where Does Data Live? — The Stack and The Heap

## ⚠️ Important Introduction

This module teaches **Ownership** — Rust's most unique and important feature. It's what makes Rust both safe and fast. Take your time with these lessons!

Many beginners find ownership confusing at first. That's normal! We'll break it down into small, digestible pieces with lots of analogies.

## The Concept: Two Different Storage Areas

Imagine you're organizing your workspace:

**Your Desk (Fast, Limited Space):**
- Items are stacked in neat piles
- Very quick to add or remove from the top
- Limited space—only for items you're using RIGHT NOW
- Everything must be a known, fixed size

**Your Warehouse (Slower, Unlimited Space):**
- Items can be any size
- Takes longer to store and retrieve
- Can grow or shrink as needed
- You get a receipt (address) to find items later

Computers work the same way. They have two places to store data:
- **The Stack** (like your desk)
- **The Heap** (like your warehouse)

## The Stack: Fast and Organized

**The Stack:**
- Stores data in a LIFO (Last In, First Out) order—like a stack of plates
- Super fast (adding/removing from the top is instant)
- Automatic cleanup (when a function ends, its stack data disappears)
- **Only for fixed-size data** (known at compile time)

**What goes on the stack:**
- Numbers: `i32`, `f64`, `bool`
- Characters: `char`
- Fixed-size arrays: `[i32; 5]`
- References (pointers to data)

**Analogy:**
Think of the stack like a stack of trays in a cafeteria. You can only take the top tray (fast!), and when you're done, you put it back on top. Everything is orderly and automatic.

## The Heap: Flexible but Slower

**The Heap:**
- Stores data that can grow or shrink
- Slower than the stack (must find available space first)
- Manual management (something must clean it up)
- For variable-size data

**What goes on the heap:**
- `String` (text that can grow)
- `Vec<T>` (lists that can grow)
- Large or dynamically-sized data

**Analogy:**
Think of the heap like a huge parking lot. When you park your car, you get a ticket with the parking spot number. Finding a spot takes time, and you need that ticket to find your car later.

## Visual Comparison

```
STACK (Fast, Fixed-Size)          HEAP (Flexible, Slower)
┌──────────────┐                  ┌─────────────────────────┐
│  number: 42  │                  │                         │
├──────────────┤                  │  "Hello, World!"        │
│  is_active   │                  │  (can grow)             │
│  true        │                  │                         │
├──────────────┤                  │  [1, 2, 3, 4, ...]      │
│  reference   │ ───────────────> │  (can grow)             │
│  0x1234      │  (points to      │                         │
└──────────────┘   heap data)     └─────────────────────────┘
```

## Code Examples

**Stack data (fixed size):**

```rust
fn main() {
    let x = 5;           // Stored on stack (i32 is fixed size)
    let y = true;        // Stored on stack (bool is fixed size)
    let z = 3.14;        // Stored on stack (f64 is fixed size)

    println!("{}, {}, {}", x, y, z);
}  // x, y, z automatically cleaned up when function ends
```

**Heap data (can grow):**

```rust
fn main() {
    let s = String::from("hello");  // "hello" stored on heap
                                     // s (a pointer) stored on stack

    // s can grow:
    let mut s = String::from("hello");
    s.push_str(", world!");  // Now "hello, world!" on heap
    println!("{}", s);
}  // Rust automatically cleans up heap memory here
```

## Why This Matters for Ownership

Here's the key insight:

**Stack data is simple:**
- Cheap to copy
- Automatic cleanup
- No complexity

**Heap data is complex:**
- Expensive to copy (the whole data must be duplicated)
- Needs explicit cleanup (or memory leaks!)
- This is where **Ownership** comes in

Rust's ownership rules exist primarily to manage heap data safely and efficiently!

## Copying vs. Moving

**Stack data (cheap to copy):**

```rust
fn main() {
    let x = 5;
    let y = x;  // Copies the value (cheap!)

    println!("x: {}, y: {}", x, y);  // Both x and y are usable
}
```

**Heap data (expensive to copy, so it MOVES instead):**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // s1 is MOVED to s2 (not copied!)

    // println!("{}", s1);  // ❌ ERROR: s1 is no longer valid!
    println!("{}", s2);     // ✅ OK: s2 owns the data now
}
```

We'll explore this more in the next lesson!

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new stack_heap_practice
cd stack_heap_practice
code .
```

### **Experiment 1: Stack Data (Numbers)**

```rust
fn main() {
    let a = 10;
    let b = a;  // Copy (cheap!)

    println!("a: {}, b: {}", a, b);  // Both work!
}
```

Run this—both `a` and `b` are usable.

### **Experiment 2: Heap Data (String)**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // Move (not copy!)

    // Uncomment the next line to see the error:
    // println!("{}", s1);  // ❌ ERROR!

    println!("{}", s2);  // ✅ OK
}
```

Run this—notice `s1` is no longer usable after the move!

### **Experiment 3: Cloning Heap Data**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();  // Explicitly copy heap data

    println!("s1: {}, s2: {}", s1, s2);  // Both work!
}
```

`.clone()` makes a deep copy of heap data (expensive but sometimes necessary).

### **Experiment 4: Function Calls**

```rust
fn main() {
    let s = String::from("hello");
    take_ownership(s);  // s is moved into the function

    // println!("{}", s);  // ❌ ERROR: s was moved!
}

fn take_ownership(some_string: String) {
    println!("{}", some_string);
}  // some_string is dropped here
```

## Key Takeaways

- ✅ **Stack**: Fast, fixed-size, automatic cleanup
- ✅ **Heap**: Flexible, variable-size, needs management
- ✅ Stack data (i32, bool, etc.) is **copied**
- ✅ Heap data (String, Vec, etc.) is **moved** by default
- ✅ Moving prevents accidental double-freeing of memory
- ✅ Use `.clone()` to explicitly copy heap data
- ✅ Ownership rules manage heap memory safely
- ✅ This is the foundation for understanding the next lessons!

**Next**: We'll learn the three ownership rules that govern how data moves around your program.
