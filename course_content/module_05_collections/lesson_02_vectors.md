# Module 5, Lesson 2: Growing Lists — Vectors (`Vec<T>`)

## The Concept: Expandable Array

Imagine a bookshelf:
- Fixed array: Shelf with exactly 5 slots (can't add more)
- Vector: Shelf that can add more slots as needed

**Vectors** are growable, heap-allocated lists.

## Creating Vectors

```rust
// Empty vector
let v: Vec<i32> = Vec::new();

// With initial values (using vec! macro)
let v = vec![1, 2, 3, 4, 5];

// With capacity (pre-allocate space)
let v: Vec<i32> = Vec::with_capacity(10);
```

## Adding Elements

```rust
let mut v = Vec::new();

v.push(1);  // Add to end
v.push(2);
v.push(3);

println!("{:?}", v);  // [1, 2, 3]
```

## Accessing Elements

```rust
let v = vec![1, 2, 3, 4, 5];

// Method 1: Indexing (panics if out of bounds)
let third = v[2];
println!("Third element: {}", third);

// Method 2: get() - returns Option (safe)
match v.get(2) {
    Some(third) => println!("Third: {}", third),
    None => println!("No third element"),
}

match v.get(10) {
    Some(val) => println!("Value: {}", val),
    None => println!("Index out of bounds"),  // This prints
}
```

## Modifying Elements

```rust
let mut v = vec![1, 2, 3];

v[0] = 10;  // Change first element

println!("{:?}", v);  // [10, 2, 3]
```

## Removing Elements

```rust
let mut v = vec![1, 2, 3, 4, 5];

let last = v.pop();  // Remove and return last
println!("Popped: {:?}", last);  // Some(5)

v.remove(1);  // Remove at index 1 (shifts elements)
println!("{:?}", v);  // [1, 3, 4]
```

## Iterating Over Vectors

```rust
let v = vec![1, 2, 3, 4, 5];

// Immutable iteration
for num in &v {
    println!("{}", num);
}

// Mutable iteration
let mut v = vec![1, 2, 3];
for num in &mut v {
    *num *= 2;
}
println!("{:?}", v);  // [2, 4, 6]

// Taking ownership
for num in v {
    println!("{}", num);
}
// v is no longer valid here
```

## Vector Methods

```rust
let mut v = vec![1, 2, 3, 4, 5];

// Length
println!("Length: {}", v.len());

// Is empty?
println!("Empty: {}", v.is_empty());

// Contains?
println!("Contains 3: {}", v.contains(&3));

// Clear all elements
v.clear();

// Capacity (allocated space)
let mut v = Vec::with_capacity(10);
v.push(1);
println!("Len: {}, Capacity: {}", v.len(), v.capacity());
```

## Vectors of Different Types

```rust
// Vector of Strings
let mut names = vec![
    String::from("Alice"),
    String::from("Bob"),
];
names.push(String::from("Charlie"));

// Vector of structs
struct Person {
    name: String,
    age: u32,
}

let people = vec![
    Person { name: String::from("Alice"), age: 30 },
    Person { name: String::from("Bob"), age: 25 },
];

// Vector of enums (store multiple types)
enum Value {
    Integer(i32),
    Float(f64),
    Text(String),
}

let values = vec![
    Value::Integer(42),
    Value::Float(3.14),
    Value::Text(String::from("hello")),
];
```

## Hands-On Practice

```bash
cargo new vector_practice
cd vector_practice
code .
```

### **Experiment 1: Basic Operations**

```rust
fn main() {
    let mut numbers = vec![1, 2, 3];

    numbers.push(4);
    numbers.push(5);

    println!("{:?}", numbers);

    if let Some(last) = numbers.pop() {
        println!("Popped: {}", last);
    }

    println!("{:?}", numbers);
}
```

### **Experiment 2: Safe Access**

```rust
fn main() {
    let v = vec![10, 20, 30];

    // Safe access
    match v.get(5) {
        Some(val) => println!("Value: {}", val),
        None => println!("Index out of bounds"),
    }

    // This would panic:
    // let val = v[5];
}
```

### **Experiment 3: Filtering**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let evens: Vec<i32> = numbers
        .iter()
        .filter(|&&n| n % 2 == 0)
        .copied()
        .collect();

    println!("Evens: {:?}", evens);
}
```

### **Challenge: Todo List**

```rust
fn main() {
    let mut todos: Vec<String> = Vec::new();

    // Add tasks
    todos.push(String::from("Learn Rust"));
    todos.push(String::from("Build a project"));
    todos.push(String::from("Contribute to open source"));

    // Display tasks
    println!("=== Todo List ===");
    for (i, task) in todos.iter().enumerate() {
        println!("{}. {}", i + 1, task);
    }

    // Complete first task
    if !todos.is_empty() {
        let completed = todos.remove(0);
        println!("\n✅ Completed: {}", completed);
    }

    // Display remaining
    println!("\n=== Remaining Tasks ===");
    for (i, task) in todos.iter().enumerate() {
        println!("{}. {}", i + 1, task);
    }
}
```

## Key Takeaways

- ✅ `Vec<T>` is a growable, heap-allocated list
- ✅ Create with `Vec::new()` or `vec![]` macro
- ✅ `.push()` adds to end, `.pop()` removes from end
- ✅ Index with `[]` (panics) or `.get()` (safe)
- ✅ Iterate with `for x in &v` (immutable) or `&mut v` (mutable)
- ✅ `.len()` for length, `.is_empty()` to check empty
- ✅ Vectors can store any type (including structs and enums)
- ✅ Use enums to store multiple types in one vector
