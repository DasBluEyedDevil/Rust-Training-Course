# Module 4, Lesson 3: Fighting the Compiler (On Purpose) — Common Ownership Errors

## The Concept: Learning from Mistakes

Imagine learning to ride a bike. You WILL fall a few times. But each fall teaches you:
- "Leaning too far left causes a fall"
- "Going too fast without braking is dangerous"
- "Looking down instead of ahead makes you unstable"

Learning Rust is similar. The compiler will stop you from making mistakes, but **each error message teaches you something important**. In this lesson, we'll intentionally make mistakes and learn from them.

Think of the compiler as a patient teacher who catches your mistakes BEFORE they become bugs.

## Error 1: Using a Value After Move

### **The Mistake**

```rust
fn main() {
    let s = String::from("hello");
    let t = s;  // s moves to t

    println!("{}", s);  // ❌ Trying to use s after it moved
}
```

### **The Error Message**

```
error[E0382]: borrow of moved value: `s`
 --> src/main.rs:5:20
  |
2 |     let s = String::from("hello");
  |         - move occurs because `s` has type `String`, which does not implement the `Copy` trait
3 |     let t = s;
  |             - value moved here
4 |
5 |     println!("{}", s);
  |                    ^ value borrowed here after move
```

### **What It Means**

The compiler is telling you:
1. Line 3: "`s` moved to `t`"
2. Line 5: "You tried to use `s` after it moved"
3. Suggestion: "String doesn't implement Copy, so it moves"

### **How to Fix**

**Option 1: Use the new owner**
```rust
let s = String::from("hello");
let t = s;
println!("{}", t);  // ✅ Use t instead
```

**Option 2: Clone if you need both**
```rust
let s = String::from("hello");
let t = s.clone();
println!("{}, {}", s, t);  // ✅ Both work
```

**Option 3: Don't move (we'll learn borrowing next lesson!)**
```rust
// Coming soon: borrowing with &
```

## Error 2: Moving in a Function and Using After

### **The Mistake**

```rust
fn main() {
    let s = String::from("hello");

    print_length(s);  // s moves into function

    println!("String was: {}", s);  // ❌ s is gone!
}

fn print_length(text: String) {
    println!("Length: {}", text.len());
}
```

### **The Error Message**

```
error[E0382]: borrow of moved value: `s`
 --> src/main.rs:6:32
  |
2 |     let s = String::from("hello");
  |         - move occurs because `s` has type `String`
3 |
4 |     print_length(s);
  |                  - value moved here
5 |
6 |     println!("String was: {}", s);
  |                                ^ value borrowed here after move
```

### **How to Fix**

**Option 1: Return ownership**
```rust
fn main() {
    let s = String::from("hello");
    let s = print_length(s);  // Get ownership back
    println!("String was: {}", s);
}

fn print_length(text: String) -> String {
    println!("Length: {}", text.len());
    text  // Return ownership
}
```

**Option 2: Clone (wasteful)**
```rust
print_length(s.clone());  // Function gets a copy
println!("String was: {}", s);  // Original still valid
```

**Option 3: Use references (best - next lesson!)**

## Error 3: Moving in a Loop

### **The Mistake**

```rust
fn main() {
    let s = String::from("hello");

    for i in 0..3 {
        print_string(s);  // ❌ Tries to move s three times!
    }
}

fn print_string(text: String) {
    println!("{}", text);
}
```

### **The Error Message**

```
error[E0382]: use of moved value: `s`
 --> src/main.rs:5:22
  |
2 |     let s = String::from("hello");
  |         - move occurs because `s` has type `String`
3 |
4 |     for i in 0..3 {
5 |         print_string(s);
  |                      ^ value moved here, in previous iteration of loop
```

### **What It Means**

- First loop iteration: `s` moves into `print_string()` and is dropped
- Second iteration: Trying to move `s` again, but it's already gone!

### **How to Fix**

**Option 1: Clone in the loop (expensive)**
```rust
for i in 0..3 {
    print_string(s.clone());
}
```

**Option 2: Use references (efficient - next lesson!)**

## Error 4: Multiple Mutable Bindings

### **The Mistake**

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;  // ❌ Two mutable references!

    println!("{}, {}", r1, r2);
}
```

### **The Error Message**

```
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 |
7 |     println!("{}, {}", r1, r2);
  |                        -- first borrow later used here
```

### **What It Means**

Rust prevents you from having multiple mutable references to the same data at the same time. This prevents data races!

### **How to Fix**

**Option 1: Don't use r1 after creating r2**
```rust
let mut s = String::from("hello");

let r1 = &mut s;
println!("{}", r1);  // Use r1

let r2 = &mut s;  // ✅ OK: r1 is no longer used
println!("{}", r2);
```

**Option 2: Use scopes**
```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;
    println!("{}", r1);
}  // r1 goes out of scope

let r2 = &mut s;  // ✅ OK: r1 is gone
println!("{}", r2);
```

## Error 5: Mixing Immutable and Mutable References

### **The Mistake**

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;      // Immutable borrow
    let r2 = &s;      // Another immutable borrow (OK)
    let r3 = &mut s;  // ❌ Mutable borrow while immutable exist!

    println!("{}, {}, {}", r1, r2, r3);
}
```

### **The Error Message**

```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:14
  |
4 |     let r1 = &s;
  |              -- immutable borrow occurs here
5 |     let r2 = &s;
6 |     let r3 = &mut s;
  |              ^^^^^^ mutable borrow occurs here
7 |
8 |     println!("{}, {}, {}", r1, r2, r3);
  |                            -- immutable borrow later used here
```

### **What It Means**

You can't modify data while others are reading it! This prevents data from changing unexpectedly.

### **How to Fix**

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
println!("{}, {}", r1, r2);  // Use immutable references

let r3 = &mut s;  // ✅ OK: r1 and r2 no longer used
println!("{}", r3);
```

## Hands-On Practice: Error Scavenger Hunt

### **Create a Practice Project**

```bash
cargo new ownership_errors
cd ownership_errors
code .
```

### **Exercise 1: Identify the Error**

What's wrong with this code?

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    let s3 = s1;  // ❌ What's the problem?
}
```

<details>
<summary>Answer</summary>

`s1` moved to `s2`, so it can't move to `s3` again.

**Fix:**
```rust
let s2 = s1.clone();
let s3 = s1.clone();
```
</details>

### **Exercise 2: Identify the Error**

```rust
fn main() {
    let s = String::from("hello");
    takes_ownership(s);
    takes_ownership(s);  // ❌ What's the problem?
}

fn takes_ownership(text: String) {
    println!("{}", text);
}
```

<details>
<summary>Answer</summary>

First call moves `s`, second call tries to move it again.

**Fix:**
```rust
takes_ownership(s.clone());
takes_ownership(s);
```

Or return ownership from the function.
</details>

### **Exercise 3: Identify the Error**

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;  // ❌ What's the problem?

    r1.push_str(" world");
    r2.push_str("!");
}
```

<details>
<summary>Answer</summary>

Can't have two mutable references at the same time.

**Fix:**
```rust
let r1 = &mut s;
r1.push_str(" world");

let r2 = &mut s;  // r1 no longer used
r2.push_str("!");
```
</details>

### **Challenge: Fix All the Errors**

This program has multiple ownership errors. Fix them WITHOUT using `.clone()`:

```rust
fn main() {
    let s = String::from("hello");

    let len = calculate_length(s);
    println!("The length of '{}' is {}", s, len);  // ❌
}

fn calculate_length(text: String) -> usize {
    text.len()
}  // text is dropped here
```

**Hint**: You'll need to return the String along with the length.

<details>
<summary>Solution</summary>

```rust
fn main() {
    let s = String::from("hello");

    let (s, len) = calculate_length(s);  // Return both!
    println!("The length of '{}' is {}", s, len);
}

fn calculate_length(text: String) -> (String, usize) {
    let length = text.len();
    (text, length)  // Return tuple
}
```

**But this is awkward!** In the next lesson, we'll learn **borrowing** - a much better solution!
</details>

## Compiler Error Patterns to Remember

1. **"borrow of moved value"** → You moved data and tried to use it again
2. **"use of moved value"** → Same as above
3. **"cannot borrow as mutable more than once"** → Multiple mutable references
4. **"cannot borrow as mutable because it is also borrowed as immutable"** → Mixed borrows

## Key Takeaways

- ✅ Compiler errors are **teaching tools**, not obstacles
- ✅ "Use after move" is the most common beginner error
- ✅ Each error message tells you EXACTLY what's wrong and where
- ✅ You can't have multiple mutable references simultaneously
- ✅ You can't mix mutable and immutable references
- ✅ Ownership errors are caught at **compile time** (no runtime crashes!)
- ✅ These restrictions prevent entire classes of bugs

## The Good News

All of these errors seem restrictive now, but they prevent:
- Segmentation faults
- Use-after-free bugs
- Data races
- Memory corruption

And in the next lesson, we'll learn **borrowing** - which makes ownership much more ergonomic!

**Next**: Learn how to **lend** data to functions without transferring ownership!
