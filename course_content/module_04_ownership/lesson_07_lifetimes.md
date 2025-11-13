# Module 4, Lesson 7: How Long Can I Use This? — Understanding Lifetimes

## The Concept: Expiration Dates

Imagine borrowing a library book. The library gives you a due date:
- You can use the book until that date
- After the due date, your borrowing privilege expires
- The library ensures the book doesn't disappear while you have it

**Lifetimes** in Rust work the same way. They track how long a reference is valid.

## What Are Lifetimes?

A **lifetime** is the scope for which a reference is valid.

**Good news**: Most of the time, Rust infers lifetimes automatically! You rarely need to write them explicitly.

```rust
fn main() {
    let s = String::from("hello");  // s's lifetime starts

    let r = &s;  // r's lifetime starts (borrows from s)

    println!("{}", r);  // Use r

}  // r's lifetime ends, then s's lifetime ends
```

Rust ensures `r` doesn't outlive `s`.

## The Lifetime Problem

What if a function returns a reference? Which input does it come from?

```rust
fn longest(x: &str, y: &str) -> &str {  // ❌ Which lifetime?
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Rust doesn't know if the returned reference is from `x` or `y`, so it doesn't know how long it's valid!

## Lifetime Annotation Syntax

We use **lifetime parameters** to tell Rust about relationships:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

**What this means:**
- `'a` is a lifetime parameter (pronounced "tick A")
- Both `x` and `y` live for at least lifetime `'a`
- The returned reference will also live for `'a`
- Rust enforces: returned reference won't outlive either input

## Lifetime Annotations Don't Change Lifetimes

**Important**: Lifetime annotations are **descriptive**, not prescriptive. They describe relationships but don't change how long things live.

Think of them like type annotations—they document relationships for the compiler.

## When Do You Need Lifetime Annotations?

### **Case 1: Multiple References Returned**

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
    let bytes = s.as_bytes();

    for (i, &byte) in bytes.iter().enumerate() {
        if byte == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

### **Case 2: Structs Holding References**

```rust
struct Book<'a> {
    title: &'a str,  // Book can't outlive this reference
    author: &'a str,
}

fn main() {
    let title = String::from("The Rust Book");
    let author = String::from("Steve Klabnik");

    let book = Book {
        title: &title,
        author: &author,
    };

    println!("{} by {}", book.title, book.author);
}  // book, author, and title all end together
```

## Lifetime Elision Rules

Rust has three rules for inferring lifetimes automatically:

1. Each input reference gets its own lifetime
2. If there's exactly one input lifetime, it's assigned to all outputs
3. If there's a `&self` or `&mut self`, its lifetime is assigned to outputs

**Most of the time, you don't need to write lifetimes!**

## The Static Lifetime

`'static` means "lives for the entire program":

```rust
let s: &'static str = "Hello, world!";  // String literals are 'static
```

## Hands-On Practice

### **Experiment 1: Lifetime Annotations**

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string");
    let string2 = String::from("short");

    let result = longest(string1.as_str(), string2.as_str());
    println!("Longest: {}", result);
}
```

### **Experiment 2: Struct with Lifetimes**

```rust
struct Article<'a> {
    title: &'a str,
    content: &'a str,
}

fn main() {
    let title = String::from("Rust Lifetimes");
    let content = String::from("Lifetimes ensure references are valid.");

    let article = Article {
        title: &title,
        content: &content,
    };

    println!("{}: {}", article.title, article.content);
}
```

### **Challenge: Safe Reference Return**

This won't compile. Fix it with lifetime annotations:

```rust
fn get_first<'a>(items: &'a [&str]) -> &'a str {
    items[0]
}

fn main() {
    let words = vec!["hello", "world"];
    let first = get_first(&words);
    println!("{}", first);
}
```

## Key Takeaways

- ✅ **Lifetimes** describe how long references are valid
- ✅ Rust infers lifetimes most of the time (lifetime elision)
- ✅ Syntax: `'a` (tick A), `'b`, `'static`
- ✅ Used when compiler can't infer reference relationships
- ✅ Lifetimes in structs ensure references don't outlive data
- ✅ `'static` means "lives for entire program"
- ✅ Lifetime annotations are **descriptive**, not prescriptive
- ✅ Most Rust code doesn't need explicit lifetime annotations!

## When You'll Use Lifetimes

- Returning references from functions with multiple inputs
- Structs that hold references
- More advanced patterns

**For now, understand the concept. You'll naturally learn when you need them through practice!**

**Next**: Practice project synthesizing all ownership concepts!
