# Module 4, Lesson 5: Lending with Permission to Change — Mutable Borrowing

## The Concept: Borrowing with Edit Rights

Imagine you lend your notebook to a friend. You have two options:

**Option 1: Read-only (immutable reference)**
- "You can look at my notes, but don't write in it"
- Multiple friends can look at it at once
- No risk of changes

**Option 2: With editing permission (mutable reference)**
- "You can look at AND edit my notes"
- Only ONE person can have it at a time (to prevent conflicts)
- You can't look at it while they're editing (to avoid seeing half-finished edits)

**Mutable references** in Rust work like Option 2. They allow modification, but with strict rules to prevent data races.

## The Problem We're Solving

What if we want a function to **modify** data without taking ownership?

```rust
fn main() {
    let mut s = String::from("hello");

    // How do we let add_world modify s without taking ownership?
    add_world(s);  // ❌ This moves s

    println!("{}", s);  // ❌ s is gone!
}

fn add_world(text: String) {
    text.push_str(" world");  // ❌ Can't modify; text is immutable
}
```

We need a way to **borrow mutably**.

## The Solution: Mutable References

Use `&mut` to create a mutable reference:

```rust
fn main() {
    let mut s = String::from("hello");

    add_world(&mut s);  // Borrow mutably

    println!("{}", s);  // ✅ Prints "hello world"
}

fn add_world(text: &mut String) {
    text.push_str(" world");  // ✅ Can modify!
}
```

**Perfect!** The function can modify the String, and we still own it afterward.

## Mutable Reference Syntax

**Creating a mutable reference: `&mut`**
```rust
let mut s = String::from("hello");
let r = &mut s;  // r is a mutable reference to s
```

**Accepting a mutable reference in a function: `&mut Type`**
```rust
fn modify_string(text: &mut String) {
    text.push_str(" more");
}
```

**Important**: The original variable MUST be `mut`!

```rust
let s = String::from("hello");  // ❌ Not mutable
let r = &mut s;  // ERROR: can't borrow immutable variable as mutable
```

```rust
let mut s = String::from("hello");  // ✅ Mutable
let r = &mut s;  // OK
```

## The Mutable Reference Rules

These are STRICT rules enforced by the compiler:

### **Rule 1: Only ONE mutable reference at a time**

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;  // ❌ ERROR! Second mutable reference

println!("{}, {}", r1, r2);
```

**Why?** If two parts of code could modify the same data simultaneously, it would cause a **data race**.

**Fix**: Use them in separate scopes

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;
    r1.push_str(" world");
}  // r1 goes out of scope

let r2 = &mut s;  // ✅ OK: r1 is gone
r2.push_str("!");
```

### **Rule 2: No mutable reference while immutable references exist**

```rust
let mut s = String::from("hello");

let r1 = &s;      // Immutable borrow
let r2 = &s;      // Another immutable borrow (OK)
let r3 = &mut s;  // ❌ ERROR! Can't have mutable while immutable exist

println!("{}, {}, {}", r1, r2, r3);
```

**Why?** Imagine reading a document while someone is editing it—you might see inconsistent data!

**Fix**: Stop using immutable references before creating a mutable one

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
println!("{}, {}", r1, r2);  // Last use of r1 and r2

let r3 = &mut s;  // ✅ OK: r1 and r2 no longer used
r3.push_str(" world");
println!("{}", r3);
```

### **Rule 3: Original variable can't be used while mutably borrowed**

```rust
let mut s = String::from("hello");

let r = &mut s;
s.push_str(" world");  // ❌ ERROR! s is borrowed
println!("{}", r);
```

**Why?** The borrow `r` would become invalid if `s` changed!

## Non-Lexical Lifetimes (NLL)

Modern Rust is smart about when borrows end:

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    println!("{}, {}", r1, r2);  // r1 and r2 last used here

    let r3 = &mut s;  // ✅ OK! r1 and r2 are no longer active
    r3.push_str(" world");
}
```

The borrow ends at the **last use**, not at the end of the scope!

## Modifying Through Mutable References

```rust
fn main() {
    let mut s = String::from("hello");

    modify(&mut s);

    println!("{}", s);  // Prints "hello world!"
}

fn modify(text: &mut String) {
    text.push_str(" world");
    text.push('!');
}
```

## Returning Mutable References

You can return mutable references, but be careful:

```rust
fn main() {
    let mut s = String::from("hello");

    let r = get_mut_ref(&mut s);
    r.push_str(" world");

    println!("{}", s);
}

fn get_mut_ref(text: &mut String) -> &mut String {
    text  // Return the mutable reference
}
```

##Mutable References to Vectors

A common pattern:

```rust
fn main() {
    let mut numbers = vec![1, 2, 3];

    add_numbers(&mut numbers);

    println!("{:?}", numbers);  // [1, 2, 3, 4, 5]
}

fn add_numbers(nums: &mut Vec<i32>) {
    nums.push(4);
    nums.push(5);
}
```

## Dereferencing Mutable References

Sometimes you need to use `*` to modify the actual value:

```rust
fn main() {
    let mut x = 5;

    add_one(&mut x);

    println!("{}", x);  // 6
}

fn add_one(num: &mut i32) {
    *num += 1;  // Dereference to modify the value
}
```

For types like `String` and `Vec`, you don't need `*` for method calls:

```rust
fn modify(text: &mut String) {
    text.push_str(" world");  // No * needed
}
```

## Common Patterns

### **Pattern 1: Modify-in-place**

```rust
fn main() {
    let mut name = String::from("alice");

    capitalize(&mut name);

    println!("{}", name);  // "Alice"
}

fn capitalize(s: &mut String) {
    if let Some(first) = s.get_mut(0..1) {
        first.make_ascii_uppercase();
    }
}
```

### **Pattern 2: Swap values**

```rust
fn main() {
    let mut a = 5;
    let mut b = 10;

    swap(&mut a, &mut b);

    println!("a: {}, b: {}", a, b);  // a: 10, b: 5
}

fn swap(x: &mut i32, y: &mut i32) {
    let temp = *x;
    *x = *y;
    *y = temp;
}
```

### **Pattern 3: Modify collection elements**

```rust
fn main() {
    let mut numbers = vec![1, 2, 3, 4, 5];

    double_values(&mut numbers);

    println!("{:?}", numbers);  // [2, 4, 6, 8, 10]
}

fn double_values(nums: &mut Vec<i32>) {
    for num in nums.iter_mut() {
        *num *= 2;
    }
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new mutable_borrowing
cd mutable_borrowing
code .
```

### **Experiment 1: Basic Mutable Reference**

```rust
fn main() {
    let mut s = String::from("hello");

    append_exclamation(&mut s);

    println!("{}", s);  // "hello!"
}

fn append_exclamation(text: &mut String) {
    text.push('!');
}
```

### **Experiment 2: Multiple Modifications**

```rust
fn main() {
    let mut s = String::from("hello");

    append_word(&mut s, "world");
    append_word(&mut s, "Rust");

    println!("{}", s);  // "hello world Rust"
}

fn append_word(text: &mut String, word: &str) {
    text.push(' ');
    text.push_str(word);
}
```

### **Experiment 3: Cannot Have Two Mutable References**

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    // let r2 = &mut s;  // ❌ Uncomment to see error

    r1.push_str(" world");
    println!("{}", r1);
}
```

### **Experiment 4: Mutable Reference Ends**

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    r1.push_str(" world");
    println!("{}", r1);  // Last use of r1

    let r2 = &mut s;  // ✅ OK: r1 no longer used
    r2.push('!');
    println!("{}", r2);
}
```

### **Challenge: String Manipulator**

Create functions that modify strings in place:

```rust
fn main() {
    let mut text = String::from("hello rust");

    to_uppercase(&mut text);
    println!("{}", text);  // "HELLO RUST"

    add_prefix(&mut text, "Language: ");
    println!("{}", text);  // "Language: HELLO RUST"

    reverse_words(&mut text);
    println!("{}", text);  // "RUST HELLO :egaugnaL"
}

fn to_uppercase(s: &mut String) {
    *s = s.to_uppercase();
}

fn add_prefix(s: &mut String, prefix: &str) {
    s.insert_str(0, prefix);
}

fn reverse_words(s: &mut String) {
    let words: Vec<&str> = s.split_whitespace().collect();
    let reversed: Vec<&str> = words.into_iter().rev().collect();
    *s = reversed.join(" ");
}
```

## Comparison: Immutable vs. Mutable References

| Feature | Immutable (`&T`) | Mutable (`&mut T`) |
|---------|------------------|---------------------|
| **Can read?** | ✅ Yes | ✅ Yes |
| **Can modify?** | ❌ No | ✅ Yes |
| **How many at once?** | ✅ Multiple | ⚠️ Only ONE |
| **Can coexist with immutable refs?** | ✅ Yes | ❌ No |
| **Syntax** | `&` | `&mut` |

## Key Takeaways

- ✅ **Mutable references** allow modification without taking ownership
- ✅ **Syntax**: `&mut` creates one, `&mut Type` accepts one
- ✅ **Only ONE mutable reference** to data at a time
- ✅ **No mutable reference** while immutable references exist
- ✅ Original variable must be declared `mut`
- ✅ Borrows end at **last use** (non-lexical lifetimes)
- ✅ Use `*` to dereference when modifying primitive types
- ✅ Prevents data races at compile time!

## The Rules Summary

```rust
let mut s = String::from("hello");

// ✅ Multiple immutable references OK
let r1 = &s;
let r2 = &s;

// ✅ One mutable reference OK (when no immutable exist)
let r3 = &mut s;

// ❌ Two mutable references NOT OK
let r4 = &mut s;  // ERROR if r3 still in use

// ❌ Mixing immutable and mutable NOT OK
let r5 = &s;      // ERROR if r3 still in use
```

**Next**: Learn common **borrow checker errors** and how to fix them!
