# Module 4, Lesson 6: Fighting the Borrow Checker — Common Borrowing Errors

## The Concept: Learning from the Borrow Checker

The **borrow checker** is Rust's compile-time guardian that enforces ownership and borrowing rules. New Rustaceans often "fight" with it, but it's actually preventing bugs!

Think of it like a strict but helpful grammar checker that catches mistakes before anyone sees your writing.

## Error 1: Multiple Mutable References

### **The Mistake**

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;  // ❌ Second mutable reference

    r1.push_str(" world");
    r2.push('!');
}
```

### **The Error**

```
error[E0499]: cannot borrow `s` as mutable more than once at a time
```

### **The Fix**

Use them in separate scopes:

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;
    r1.push_str(" world");
}  // r1 ends here

let r2 = &mut s;
r2.push('!');
```

## Error 2: Mixing Immutable and Mutable References

### **The Mistake**

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &mut s;  // ❌ Mutable while immutable exists

println!("{}, {}", r1, r2);
```

### **The Fix**

```rust
let mut s = String::from("hello");

let r1 = &s;
println!("{}", r1);  // Last use of r1

let r2 = &mut s;  // ✅ OK now
r2.push_str(" world");
```

## Error 3: Dangling References

### **The Mistake**

```rust
fn main() {
    let r = dangle();
}

fn dangle() -> &String {  // ❌ Returns reference to local variable
    let s = String::from("hello");
    &s  // s is dropped at end of function!
}
```

### **The Fix**

Return ownership instead:

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s  // ✅ Return ownership
}
```

## Error 4: Modifying While Borrowed

### **The Mistake**

```rust
let mut v = vec![1, 2, 3];

let first = &v[0];
v.push(4);  // ❌ Modifying while borrowed

println!("{}", first);
```

### **The Fix**

```rust
let mut v = vec![1, 2, 3];

let first = v[0];  // Copy the value
v.push(4);  // ✅ OK

println!("{}", first);
```

## Hands-On Practice

### **Exercise 1: Fix the Borrowing Error**

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    let r3 = &mut s;  // ❌ Fix this

    println!("{}, {}, {}", r1, r2, r3);
}
```

<details>
<summary>Solution</summary>

```rust
let r1 = &s;
let r2 = &s;
println!("{}, {}", r1, r2);  // Use immutable refs

let r3 = &mut s;  // ✅ OK now
println!("{}", r3);
```
</details>

### **Exercise 2: Fix the Double Mutable Borrow**

```rust
fn main() {
    let mut numbers = vec![1, 2, 3];

    let r1 = &mut numbers;
    let r2 = &mut numbers;  // ❌ Fix this

    r1.push(4);
    r2.push(5);
}
```

<details>
<summary>Solution</summary>

```rust
let mut numbers = vec![1, 2, 3];

{
    let r1 = &mut numbers;
    r1.push(4);
}  // r1 ends

let r2 = &mut numbers;
r2.push(5);
```
</details>

## Key Takeaways

- ✅ Borrow checker errors are **preventing bugs**, not being difficult
- ✅ Most errors solved by understanding reference lifetimes
- ✅ Use scopes to limit borrow duration
- ✅ End immutable borrows before creating mutable ones
- ✅ Copy values if you need them after modification
- ✅ Never return references to local variables
- ✅ Trust the compiler—it's usually right!

**Next**: Understanding **lifetimes** - how long references are valid!
