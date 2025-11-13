# Module 8, Lesson 3: Essential Behaviors — Common Traits in the Standard Library

## The Concept: Standard Interfaces

Imagine electronic devices with standard ports:
- **USB-C**: Charging, data transfer, video
- **3.5mm jack**: Audio
- **HDMI**: Video and audio

**Common traits** provide standard interfaces that make Rust types work with built-in language features.

## Why Common Traits Matter

They unlock built-in functionality:
- **Debug**: `println!("{:?}", value)`
- **Display**: `println!("{}", value)`
- **Clone**: `value.clone()`
- **Copy**: Automatic copying
- **PartialEq**: `a == b`
- **Ord**: Sorting and comparison

## Debug - For Developer Output

```rust
#[derive(Debug)]
struct User {
    name: String,
    age: u32,
}

fn main() {
    let user = User {
        name: String::from("Alice"),
        age: 30,
    };

    println!("{:?}", user);      // User { name: "Alice", age: 30 }
    println!("{:#?}", user);     // Pretty-printed
}
```

**Output:**
```
User { name: "Alice", age: 30 }
User {
    name: "Alice",
    age: 30,
}
```

### **Custom Debug Implementation**

```rust
use std::fmt;

struct Point {
    x: i32,
    y: i32,
}

impl fmt::Debug for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Point({}, {})", self.x, self.y)
    }
}

fn main() {
    let p = Point { x: 10, y: 20 };
    println!("{:?}", p);  // Point(10, 20)
}
```

## Display - For User-Facing Output

```rust
use std::fmt;

struct User {
    name: String,
    age: u32,
}

impl fmt::Display for User {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{} (age {})", self.name, self.age)
    }
}

fn main() {
    let user = User {
        name: String::from("Alice"),
        age: 30,
    };

    println!("{}", user);  // Alice (age 30)
}
```

## Clone - Explicit Copying

```rust
#[derive(Debug, Clone)]
struct Book {
    title: String,
    pages: u32,
}

fn main() {
    let book1 = Book {
        title: String::from("Rust Book"),
        pages: 500,
    };

    let book2 = book1.clone();  // Explicit copy

    println!("Book 1: {:?}", book1);
    println!("Book 2: {:?}", book2);
    // Both are valid!
}
```

### **Custom Clone Implementation**

```rust
#[derive(Debug)]
struct Counter {
    count: u32,
}

impl Clone for Counter {
    fn clone(&self) -> Self {
        println!("Cloning counter with count {}", self.count);
        Counter { count: self.count }
    }
}

fn main() {
    let c1 = Counter { count: 10 };
    let c2 = c1.clone();
    println!("{:?}", c2);
}
```

## Copy - Automatic Copying

```rust
#[derive(Debug, Copy, Clone)]  // Copy requires Clone
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p1 = Point { x: 5, y: 10 };
    let p2 = p1;  // Automatic copy, both valid

    println!("p1: {:?}", p1);
    println!("p2: {:?}", p2);
}
```

**Rules for Copy:**
- Type must be fully stored on the stack
- No heap allocations (no `String`, `Vec`, etc.)
- All fields must implement `Copy`

```rust
// ❌ This won't compile: String doesn't implement Copy
// #[derive(Copy, Clone)]
// struct User {
//     name: String,
// }

// ✅ This works: i32 implements Copy
#[derive(Debug, Copy, Clone)]
struct Coordinates {
    x: i32,
    y: i32,
    z: i32,
}
```

## PartialEq and Eq - Equality Comparison

```rust
#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p1 = Point { x: 5, y: 10 };
    let p2 = Point { x: 5, y: 10 };
    let p3 = Point { x: 3, y: 7 };

    println!("p1 == p2: {}", p1 == p2);  // true
    println!("p1 == p3: {}", p1 == p3);  // false
    println!("p1 != p3: {}", p1 != p3);  // true
}
```

### **Custom PartialEq**

```rust
struct CaseInsensitiveString {
    value: String,
}

impl PartialEq for CaseInsensitiveString {
    fn eq(&self, other: &Self) -> bool {
        self.value.to_lowercase() == other.value.to_lowercase()
    }
}

fn main() {
    let s1 = CaseInsensitiveString {
        value: String::from("Hello"),
    };
    let s2 = CaseInsensitiveString {
        value: String::from("HELLO"),
    };

    println!("Equal: {}", s1 == s2);  // true
}
```

## PartialOrd and Ord - Ordering

```rust
#[derive(Debug, PartialEq, PartialOrd)]
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let alice = Person {
        name: String::from("Alice"),
        age: 30,
    };
    let bob = Person {
        name: String::from("Bob"),
        age: 25,
    };

    if alice > bob {
        println!("Alice is greater");
    } else {
        println!("Bob is greater");
    }
}
```

### **Custom Ordering**

```rust
use std::cmp::Ordering;

#[derive(Debug, Eq, PartialEq)]
struct Student {
    name: String,
    grade: u32,
}

impl PartialOrd for Student {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        Some(self.cmp(other))
    }
}

impl Ord for Student {
    fn cmp(&self, other: &Self) -> Ordering {
        // Sort by grade descending
        other.grade.cmp(&self.grade)
    }
}

fn main() {
    let mut students = vec![
        Student {
            name: String::from("Alice"),
            grade: 85,
        },
        Student {
            name: String::from("Bob"),
            grade: 92,
        },
        Student {
            name: String::from("Charlie"),
            grade: 78,
        },
    ];

    students.sort();

    for student in students {
        println!("{}: {}", student.name, student.grade);
    }
}
```

**Output:**
```
Bob: 92
Alice: 85
Charlie: 78
```

## Default - Default Values

```rust
#[derive(Debug, Default)]
struct Config {
    debug: bool,
    max_connections: u32,
    timeout: u32,
}

fn main() {
    let config = Config::default();
    println!("{:?}", config);
    // Config { debug: false, max_connections: 0, timeout: 0 }

    let custom = Config {
        debug: true,
        ..Default::default()  // Use defaults for other fields
    };
    println!("{:?}", custom);
}
```

### **Custom Default**

```rust
struct Server {
    host: String,
    port: u16,
}

impl Default for Server {
    fn default() -> Self {
        Server {
            host: String::from("localhost"),
            port: 8080,
        }
    }
}

fn main() {
    let server = Server::default();
    println!("Server: {}:{}", server.host, server.port);
}
```

## From and Into - Type Conversion

```rust
struct Celsius(f64);
struct Fahrenheit(f64);

impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self {
        Fahrenheit(c.0 * 9.0 / 5.0 + 32.0)
    }
}

fn main() {
    let celsius = Celsius(25.0);
    let fahrenheit: Fahrenheit = celsius.into();  // into() is automatic!

    println!("Temperature: {}°F", fahrenheit.0);
}
```

## Hands-On Practice

```bash
cargo new common_traits
cd common_traits
code .
```

### **Experiment 1: Complete Type**

```rust
use std::fmt;

#[derive(Debug, Clone, PartialEq)]
struct Product {
    id: u32,
    name: String,
    price: f64,
}

impl fmt::Display for Product {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{} (ID: {}) - ${:.2}", self.name, self.id, self.price)
    }
}

impl Default for Product {
    fn default() -> Self {
        Product {
            id: 0,
            name: String::from("Unknown"),
            price: 0.0,
        }
    }
}

fn main() {
    let product1 = Product {
        id: 1,
        name: String::from("Laptop"),
        price: 999.99,
    };

    let product2 = product1.clone();

    println!("Debug: {:?}", product1);
    println!("Display: {}", product1);
    println!("Equal: {}", product1 == product2);

    let default_product = Product::default();
    println!("Default: {}", default_product);
}
```

### **Experiment 2: Sortable Items**

```rust
#[derive(Debug, Eq, PartialEq)]
struct Task {
    name: String,
    priority: u32,
}

impl PartialOrd for Task {
    fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
        Some(self.cmp(other))
    }
}

impl Ord for Task {
    fn cmp(&self, other: &Self) -> std::cmp::Ordering {
        // High priority first
        other.priority.cmp(&self.priority)
    }
}

fn main() {
    let mut tasks = vec![
        Task {
            name: String::from("Email client"),
            priority: 2,
        },
        Task {
            name: String::from("Fix bug"),
            priority: 5,
        },
        Task {
            name: String::from("Write docs"),
            priority: 1,
        },
    ];

    tasks.sort();

    println!("Tasks by priority:");
    for task in tasks {
        println!("  [P{}] {}", task.priority, task.name);
    }
}
```

### **Challenge: Complete Contact System**

```rust
use std::fmt;

#[derive(Debug, Clone, PartialEq, Eq)]
struct Contact {
    name: String,
    email: String,
    phone: String,
}

impl fmt::Display for Contact {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{} <{}> | {}", self.name, self.email, self.phone)
    }
}

impl PartialOrd for Contact {
    fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
        Some(self.cmp(other))
    }
}

impl Ord for Contact {
    fn cmp(&self, other: &Self) -> std::cmp::Ordering {
        // Sort alphabetically by name
        self.name.cmp(&other.name)
    }
}

impl Default for Contact {
    fn default() -> Self {
        Contact {
            name: String::from("Unknown"),
            email: String::from("no-email"),
            phone: String::from("000-000-0000"),
        }
    }
}

fn main() {
    let mut contacts = vec![
        Contact {
            name: String::from("Charlie"),
            email: String::from("charlie@example.com"),
            phone: String::from("555-3333"),
        },
        Contact {
            name: String::from("Alice"),
            email: String::from("alice@example.com"),
            phone: String::from("555-1111"),
        },
        Contact {
            name: String::from("Bob"),
            email: String::from("bob@example.com"),
            phone: String::from("555-2222"),
        },
    ];

    println!("=== Unsorted Contacts ===");
    for contact in &contacts {
        println!("{}", contact);
    }

    contacts.sort();

    println!("\n=== Sorted Contacts ===");
    for contact in &contacts {
        println!("{}", contact);
    }

    let contact2 = contacts[0].clone();
    println!("\n=== Cloned Contact ===");
    println!("{}", contact2);

    println!("\n=== Comparison ===");
    println!("First == Cloned: {}", contacts[0] == contact2);

    println!("\n=== Default Contact ===");
    let default = Contact::default();
    println!("{:?}", default);
}
```

**Output:**
```
=== Unsorted Contacts ===
Charlie <charlie@example.com> | 555-3333
Alice <alice@example.com> | 555-1111
Bob <bob@example.com> | 555-2222

=== Sorted Contacts ===
Alice <alice@example.com> | 555-1111
Bob <bob@example.com> | 555-2222
Charlie <charlie@example.com> | 555-3333

=== Cloned Contact ===
Alice <alice@example.com> | 555-1111

=== Comparison ===
First == Cloned: true

=== Default Contact ===
Contact { name: "Unknown", email: "no-email", phone: "000-000-0000" }
```

## Derivable vs Manual Implementation

### **Derivable (Use `#[derive(...)]`)**

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Default)]
struct Point {
    x: i32,
    y: i32,
}
```

### **Manual (Custom Behavior)**

```rust
impl PartialEq for Point {
    fn eq(&self, other: &Self) -> bool {
        // Custom equality logic
        self.x == other.x && self.y == other.y
    }
}
```

## Trait Summary Table

| Trait | Purpose | Enables | Derivable? |
|-------|---------|---------|------------|
| `Debug` | Developer output | `{:?}` formatting | ✅ Yes |
| `Display` | User output | `{}` formatting | ❌ No |
| `Clone` | Explicit copying | `.clone()` | ✅ Yes |
| `Copy` | Implicit copying | Automatic copy | ✅ Yes |
| `PartialEq` | Equality | `==`, `!=` | ✅ Yes |
| `Eq` | Total equality | Required for HashMap keys | ✅ Yes |
| `PartialOrd` | Partial ordering | `<`, `>`, `<=`, `>=` | ✅ Yes |
| `Ord` | Total ordering | `.sort()` | ✅ Yes |
| `Default` | Default values | `T::default()` | ✅ Yes |
| `From/Into` | Type conversion | `.into()` | ❌ No |

## Best Practices

### **1. Derive When Possible**

```rust
// ✅ Good
#[derive(Debug, Clone, PartialEq)]
struct User {
    name: String,
}

// ❌ Don't manually implement unless needed
```

### **2. Implement Display for User-Facing Types**

```rust
impl fmt::Display for User {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "User: {}", self.name)
    }
}
```

### **3. Copy Only for Small, Stack-Only Types**

```rust
// ✅ Good for Copy
#[derive(Copy, Clone)]
struct Point { x: i32, y: i32 }

// ❌ Bad for Copy (has heap data)
// struct User { name: String }
```

## Key Takeaways

- ✅ Common traits provide standard interfaces
- ✅ Use `#[derive(...)]` for automatic implementations
- ✅ `Debug` is for developers, `Display` for users
- ✅ `Clone` is explicit, `Copy` is implicit (automatic)
- ✅ `Copy` types must be stack-only (no heap)
- ✅ `PartialEq` enables `==`, `Ord` enables sorting
- ✅ `Default` provides sensible initial values
- ✅ `From`/`Into` enable type conversions
- ✅ Manual implementation for custom behavior
- ✅ Traits unlock built-in Rust functionality

**Next**: Iterators for powerful data processing!

---

**Progress**: Module 8, Lesson 3 complete (46/60 lessons total)
