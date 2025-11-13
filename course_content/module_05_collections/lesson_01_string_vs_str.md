# Module 5, Lesson 1: Text You Own vs. Text You Borrow — `String` vs `&str`

## The Concept: Owned Book vs. Page Reference

Imagine two ways to have text:
- **Owning a book**: You can add pages, remove pages, modify it (like `String`)
- **Pointing to a page**: You can read it but not modify the book (like `&str`)

## What is String?

`String` is an **owned**, **growable**, **UTF-8** text stored on the heap.

```rust
let mut s = String::from("hello");
s.push_str(" world");  // Can grow!
println!("{}", s);  // "hello world"
```

**Characteristics:**
- Owned (you can modify it)
- Stored on heap
- Can grow and shrink
- Automatically freed when owner goes out of scope

## What is &str?

`&str` (string slice) is a **view** into string data.

```rust
let s = "hello";  // &str (string literal)
let s2 = &String::from("world")[..];  // &str (slice of String)
```

**Characteristics:**
- Borrowed (read-only)
- Fixed size
- Can point to String, string literal, or part of a string
- Lightweight (just a pointer + length)

## Creating Strings

```rust
// String literals are &str
let s1 = "hello";  // Type: &str

// Create owned String
let s2 = String::from("hello");  // Type: String
let s3 = "hello".to_string();    // Type: String

// From format! macro
let s4 = format!("Number: {}", 42);  // Type: String
```

## String Methods

```rust
let mut s = String::from("hello");

// Append
s.push_str(" world");  // Add &str
s.push('!');           // Add char

// Modify
s = s.replace("world", "Rust");

// Capacity management
s.reserve(10);  // Reserve space
s.shrink_to_fit();  // Free unused space

println!("{}", s);  // "hello Rust!"
```

## String Slices (&str)

```rust
let s = String::from("hello world");

let hello = &s[0..5];    // "hello"
let world = &s[6..11];   // "world"
let all = &s[..];        // "hello world"

println!("{}, {}", hello, world);
```

## When to Use Each

**Use `String` when:**
- You need to own the text
- Text will grow or shrink
- Building strings dynamically
- Storing in structs

**Use `&str` when:**
- Reading/viewing text
- Function parameters (most flexible)
- String literals
- Efficiency (no allocation)

## Function Parameters: Prefer &str

```rust
// ✅ Good: Accepts both String and &str
fn print_text(s: &str) {
    println!("{}", s);
}

fn main() {
    let owned = String::from("hello");
    let borrowed = "world";

    print_text(&owned);     // Works
    print_text(borrowed);   // Works
}
```

```rust
// ❌ Less flexible: Only accepts String
fn print_text_owned(s: String) {
    println!("{}", s);
}

fn main() {
    let s = "hello";
    print_text_owned(s.to_string());  // Must convert
}
```

## Concatenation

```rust
// Method 1: + operator (takes ownership)
let s1 = String::from("Hello");
let s2 = String::from(" world");
let s3 = s1 + &s2;  // s1 is moved
// println!("{}", s1);  // ❌ Error

// Method 2: format! macro (doesn't take ownership)
let s1 = String::from("Hello");
let s2 = String::from(" world");
let s3 = format!("{}{}", s1, s2);
println!("{}, {}, {}", s1, s2, s3);  // ✅ All still valid

// Method 3: push_str (mutable)
let mut s = String::from("Hello");
s.push_str(" world");
```

## Hands-On Practice

```bash
cargo new string_practice
cd string_practice
code .
```

### **Experiment 1: Creating Strings**

```rust
fn main() {
    let literal = "Hello";  // &str
    let owned = String::from("World");  // String

    println!("{}, {}", literal, owned);
}
```

### **Experiment 2: Growing Strings**

```rust
fn main() {
    let mut s = String::new();

    s.push_str("Hello");
    s.push(' ');
    s.push_str("Rust");

    println!("{}", s);  // "Hello Rust"
}
```

### **Experiment 3: String Slices**

```rust
fn main() {
    let s = String::from("Hello, world!");

    let hello = &s[0..5];
    let world = &s[7..12];

    println!("{}, {}", hello, world);
}
```

### **Challenge: String Builder**

```rust
fn build_sentence(words: Vec<&str>) -> String {
    words.join(" ")
}

fn main() {
    let words = vec!["Rust", "is", "awesome"];
    let sentence = build_sentence(words);

    println!("{}", sentence);
}
```

## Key Takeaways

- ✅ `String` = owned, growable, heap-allocated
- ✅ `&str` = borrowed view, fixed size, efficient
- ✅ String literals are `&str`
- ✅ Prefer `&str` for function parameters
- ✅ Use `.to_string()` or `String::from()` to convert
- ✅ Slicing creates `&str` from `String`
- ✅ `format!` doesn't take ownership
- ✅ `+` operator takes ownership of left operand
