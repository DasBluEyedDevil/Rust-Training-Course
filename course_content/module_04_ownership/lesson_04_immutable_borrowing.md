# Module 4, Lesson 4: Lending Data Temporarily — Immutable Borrowing (References)

## The Concept: Borrowing a Book

Imagine you own a book. Your friend wants to read it. You have two options:

**Option 1: Give them the book (transfer ownership)**
- They take the book home
- You can't read it until they give it back
- When they're done, they might forget to return it!

**Option 2: Let them borrow it (reference)**
- They can read it at your house
- You still own it
- They can't write in it or damage it
- When they're done, you automatically have it back

**Borrowing** in Rust works like Option 2. You let a function "look at" your data without giving away ownership.

## The Problem We're Solving

Remember this awkward code from the last lesson?

```rust
fn main() {
    let s = String::from("hello");
    let (s, len) = calculate_length(s);  // Have to return s back!
    println!("Length of '{}' is {}", s, len);
}

fn calculate_length(text: String) -> (String, usize) {
    let length = text.len();
    (text, length)  // Return both!
}
```

This is clunky! We just want to **read** the length, not take ownership.

## The Solution: References

A **reference** lets you refer to a value without owning it:

```rust
fn main() {
    let s = String::from("hello");

    let len = calculate_length(&s);  // Borrow s with &

    println!("Length of '{}' is {}", s, len);  // s is still valid!
}

fn calculate_length(text: &String) -> usize {
    text.len()
}  // text goes out of scope, but it doesn't own the String, so nothing is dropped
```

**Much better!** Now we can use `s` after calling the function.

## Reference Syntax

**Creating a reference: `&`**
```rust
let s = String::from("hello");
let r = &s;  // r is a reference to s
```

**Accepting a reference in a function: `&Type`**
```rust
fn print_string(text: &String) {
    println!("{}", text);
}
```

**Visual representation:**

```
Stack:                    Heap:
┌──────────┐
│ s        │ ───────────> "hello"
└──────────┘              ↑
┌──────────┐              │
│ r        │ ──────────────┘ (points to the same data)
└──────────┘
```

`r` doesn't own "hello"—it just points to it!

## Borrowing Rules for Immutable References

1. **You can read the data**
   ```rust
   let s = String::from("hello");
   let r = &s;
   println!("{}", r);  // ✅ Reading is OK
   ```

2. **You CANNOT modify the data**
   ```rust
   let s = String::from("hello");
   let r = &s;
   r.push_str(" world");  // ❌ ERROR! Can't modify through immutable reference
   ```

3. **You can have MULTIPLE immutable references**
   ```rust
   let s = String::from("hello");
   let r1 = &s;
   let r2 = &s;
   let r3 = &s;
   println!("{}, {}, {}", r1, r2, r3);  // ✅ All OK!
   ```

4. **Original owner can't modify while borrowed**
   ```rust
   let mut s = String::from("hello");
   let r = &s;
   s.push_str(" world");  // ❌ ERROR! s is borrowed
   println!("{}", r);
   ```

## Multiple Immutable References (Allowed!)

You can have as many "readers" as you want:

```rust
fn main() {
    let s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    let r3 = &s;

    println!("{}, {}, {}", r1, r2, r3);  // ✅ All fine!
}
```

**Why is this safe?** Everyone is just **reading**. No one can modify the data, so there's no risk of conflict.

## Dereferencing (Usually Automatic)

To access the actual value, you can use `*` (dereference):

```rust
let x = 5;
let r = &x;

println!("{}", *r);  // Dereference to get 5
```

But Rust usually does this automatically for you:

```rust
let s = String::from("hello");
let r = &s;

println!("{}", r.len());  // ✅ Rust auto-dereferences for method calls
```

## Functions with References

**Passing a reference:**

```rust
fn main() {
    let s = String::from("hello");

    print_length(&s);  // Pass a reference

    println!("{}", s);  // ✅ s still valid!
}

fn print_length(text: &String) {
    println!("Length: {}", text.len());
}  // text (the reference) goes out of scope, but the String is not dropped
```

**Multiple parameters:**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = String::from("world");

    let result = concatenate(&s1, &s2);

    println!("{}", result);
    println!("Still have: {} and {}", s1, s2);  // ✅ Both still valid
}

fn concatenate(a: &String, b: &String) -> String {
    format!("{} {}", a, b)  // Creates a NEW String
}
```

## References to References

You can have references to references:

```rust
fn main() {
    let s = String::from("hello");
    let r1 = &s;
    let r2 = &r1;  // Reference to a reference
    let r3 = &r2;

    println!("{}", r3);  // Rust auto-dereferences all the way down
}
```

But this is rarely needed in practice.

## Common Patterns

### **Pattern 1: Reading without taking ownership**

```rust
fn get_first_word(text: &String) -> &str {
    let bytes = text.as_bytes();

    for (i, &byte) in bytes.iter().enumerate() {
        if byte == b' ' {
            return &text[0..i];
        }
    }

    &text[..]
}

fn main() {
    let sentence = String::from("hello world");
    let word = get_first_word(&sentence);

    println!("First word: {}", word);
    println!("Full sentence: {}", sentence);  // ✅ Still valid
}
```

### **Pattern 2: Multiple reads from different functions**

```rust
fn main() {
    let data = String::from("important data");

    validate(&data);
    process(&data);
    display(&data);

    println!("Original data: {}", data);  // ✅ Still valid
}

fn validate(text: &String) {
    println!("Validating: {}", text);
}

fn process(text: &String) {
    println!("Processing: {}", text);
}

fn display(text: &String) {
    println!("Displaying: {}", text);
}
```

### **Pattern 3: Iterating without consuming**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];

    print_sum(&numbers);
    print_average(&numbers);

    println!("Numbers: {:?}", numbers);  // ✅ Still valid
}

fn print_sum(nums: &Vec<i32>) {
    let sum: i32 = nums.iter().sum();
    println!("Sum: {}", sum);
}

fn print_average(nums: &Vec<i32>) {
    let sum: i32 = nums.iter().sum();
    let avg = sum as f64 / nums.len() as f64;
    println!("Average: {}", avg);
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new immutable_borrowing
cd immutable_borrowing
code .
```

### **Experiment 1: Basic References**

```rust
fn main() {
    let s = String::from("hello");

    let len = calculate_length(&s);

    println!("The length of '{}' is {}", s, len);
}

fn calculate_length(text: &String) -> usize {
    text.len()
}
```

### **Experiment 2: Multiple References**

```rust
fn main() {
    let s = String::from("Rust");

    let r1 = &s;
    let r2 = &s;
    let r3 = &s;

    println!("{}, {}, {}", r1, r2, r3);  // All work!
}
```

### **Experiment 3: References in Loops**

```rust
fn main() {
    let s = String::from("hello");

    for i in 0..5 {
        print_string(&s);  // Borrow each time
    }

    println!("Original: {}", s);  // ✅ Still valid
}

fn print_string(text: &String) {
    println!("{}", text);
}
```

### **Experiment 4: Cannot Modify Through Immutable Reference**

```rust
fn main() {
    let s = String::from("hello");
    let r = &s;

    // Uncomment to see the error:
    // r.push_str(" world");  // ❌ ERROR!

    println!("{}", r);
}
```

### **Challenge: String Analyzer**

Create functions that analyze a string without taking ownership:

```rust
fn main() {
    let text = String::from("Hello, Rust programming!");

    println!("Length: {}", get_length(&text));
    println!("Word count: {}", count_words(&text));
    println!("Contains 'Rust': {}", contains_word(&text, "Rust"));

    println!("Original text: {}", text);  // ✅ Still valid!
}

fn get_length(s: &String) -> usize {
    s.len()
}

fn count_words(s: &String) -> usize {
    s.split_whitespace().count()
}

fn contains_word(s: &String, word: &str) -> bool {
    s.contains(word)
}
```

## When to Use References

✅ **Use references when:**
- You need to read data without taking ownership
- Multiple parts of code need to read the same data
- You want to avoid expensive clones
- You're implementing a function that just observes data

❌ **Don't use references when:**
- The function needs to take ownership (e.g., to return it or store it)
- You need to modify the data (use mutable references - next lesson!)

## Key Takeaways

- ✅ **References** let you borrow data without taking ownership
- ✅ **Syntax**: `&` creates a reference, `&Type` accepts one
- ✅ **Immutable references** allow reading but NOT writing
- ✅ **Multiple immutable references** are allowed simultaneously
- ✅ Original data cannot be modified while borrowed immutably
- ✅ References are automatically dereferenced for method calls
- ✅ When the reference goes out of scope, the borrow ends
- ✅ Much more ergonomic than transferring ownership back and forth!

## The Analogy Recap

| Concept | Book Analogy |
|---------|--------------|
| Ownership | You own the book |
| Move | Give the book away |
| Immutable reference | Let someone read it (but not write in it) |
| Multiple immutable refs | Multiple people can read it |
| Reference goes out of scope | Person done reading, book back to you |

**Next**: Learn about **mutable references** - borrowing with permission to modify!
