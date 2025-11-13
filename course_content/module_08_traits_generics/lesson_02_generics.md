# Module 8, Lesson 2: Writing Flexible Code — Introduction to Generics

## The Concept: Universal Remote

Imagine a universal remote:
- **Works with**: TV, stereo, DVD player, gaming console
- **Same buttons**: Power, volume, channel
- **One design**: Adapts to many devices

**Generics** let you write code that works with many types without duplication.

## The Problem: Code Duplication

Without generics, you'd need separate functions for each type:

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn largest_f64(list: &[f64]) -> f64 {
    let mut largest = list[0];
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

// Same logic, different types! 🤦
```

## The Solution: Generic Functions

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];
    let result = largest(&numbers);
    println!("Largest number: {}", result);

    let floats = vec![1.5, 3.2, 2.8, 4.1];
    let result = largest(&floats);
    println!("Largest float: {}", result);
}
```

**Output:**
```
Largest number: 100
Largest float: 4.1
```

## Generic Syntax Explained

```rust
fn function_name<T>(parameter: T) -> T {
    // T is a type parameter
    // It's a placeholder for any type
}
```

- **`<T>`**: Type parameter declaration
- **`T`**: Convention (can be any name, but T is standard)
- **Trait bounds**: `T: PartialOrd + Copy` - T must implement these traits

## Generic Structs

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer_point = Point { x: 5, y: 10 };
    let float_point = Point { x: 1.5, y: 4.7 };

    println!("Integer point: ({}, {})", integer_point.x, integer_point.y);
    println!("Float point: ({}, {})", float_point.x, float_point.y);
}
```

### **Multiple Type Parameters**

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let mixed = Point { x: 5, y: 4.5 };  // i32 and f64
    let both_int = Point { x: 1, y: 2 };  // both i32
    let both_float = Point { x: 1.0, y: 2.0 };  // both f64

    println!("Mixed: ({}, {})", mixed.x, mixed.y);
}
```

## Generic Enums

Rust's `Option` and `Result` are generic enums!

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### **Custom Generic Enum**

```rust
enum Container<T> {
    Single(T),
    Multiple(Vec<T>),
    Empty,
}

fn main() {
    let one = Container::Single(42);
    let many = Container::Multiple(vec![1, 2, 3, 4, 5]);
    let none: Container<i32> = Container::Empty;

    match one {
        Container::Single(value) => println!("Single value: {}", value),
        Container::Multiple(values) => println!("Multiple: {:?}", values),
        Container::Empty => println!("Empty"),
    }
}
```

## Generic Methods

```rust
struct Pair<T> {
    first: T,
    second: T,
}

impl<T> Pair<T> {
    fn new(first: T, second: T) -> Self {
        Pair { first, second }
    }

    fn swap(self) -> Self {
        Pair {
            first: self.second,
            second: self.first,
        }
    }
}

// Methods only for types that implement Display + PartialOrd
impl<T: std::fmt::Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.first >= self.second {
            println!("First is larger or equal: {}", self.first);
        } else {
            println!("Second is larger: {}", self.second);
        }
    }
}

fn main() {
    let pair = Pair::new(10, 20);
    pair.cmp_display();  // Second is larger: 20

    let swapped = pair.swap();
    swapped.cmp_display();  // First is larger or equal: 20
}
```

## Trait Bounds in Detail

### **Single Trait Bound**

```rust
fn print_value<T: std::fmt::Display>(value: T) {
    println!("Value: {}", value);
}
```

### **Multiple Trait Bounds**

```rust
fn process<T: std::fmt::Display + Clone>(value: T) {
    println!("Value: {}", value);
    let copy = value.clone();
    println!("Copy: {}", copy);
}
```

### **Where Clauses (More Readable)**

```rust
fn complex_function<T, U>(t: T, u: U) -> String
where
    T: std::fmt::Display + Clone,
    U: std::fmt::Debug + Clone,
{
    format!("T: {}, U: {:?}", t, u)
}
```

## Hands-On Practice

```bash
cargo new generics_practice
cd generics_practice
code .
```

### **Experiment 1: Generic Container**

```rust
struct Box<T> {
    value: T,
}

impl<T> Box<T> {
    fn new(value: T) -> Self {
        Box { value }
    }

    fn get(&self) -> &T {
        &self.value
    }

    fn set(&mut self, value: T) {
        self.value = value;
    }
}

fn main() {
    let mut int_box = Box::new(42);
    println!("Integer box: {}", int_box.get());

    int_box.set(100);
    println!("Updated: {}", int_box.get());

    let string_box = Box::new(String::from("Hello"));
    println!("String box: {}", string_box.get());
}
```

### **Experiment 2: Generic Pair Operations**

```rust
struct Pair<T> {
    first: T,
    second: T,
}

impl<T: PartialOrd> Pair<T> {
    fn new(first: T, second: T) -> Self {
        Pair { first, second }
    }

    fn larger(&self) -> &T {
        if self.first > self.second {
            &self.first
        } else {
            &self.second
        }
    }
}

impl<T: std::ops::Add<Output = T> + Copy> Pair<T> {
    fn sum(&self) -> T {
        self.first + self.second
    }
}

fn main() {
    let number_pair = Pair::new(10, 25);
    println!("Larger: {}", number_pair.larger());
    println!("Sum: {}", number_pair.sum());

    let float_pair = Pair::new(3.5, 2.1);
    println!("Larger: {}", float_pair.larger());
    println!("Sum: {}", float_pair.sum());
}
```

### **Experiment 3: Generic Stack**

```rust
struct Stack<T> {
    items: Vec<T>,
}

impl<T> Stack<T> {
    fn new() -> Self {
        Stack { items: Vec::new() }
    }

    fn push(&mut self, item: T) {
        self.items.push(item);
    }

    fn pop(&mut self) -> Option<T> {
        self.items.pop()
    }

    fn peek(&self) -> Option<&T> {
        self.items.last()
    }

    fn is_empty(&self) -> bool {
        self.items.is_empty()
    }

    fn size(&self) -> usize {
        self.items.len()
    }
}

fn main() {
    let mut stack = Stack::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);

    println!("Stack size: {}", stack.size());
    println!("Top item: {:?}", stack.peek());

    while let Some(item) = stack.pop() {
        println!("Popped: {}", item);
    }

    println!("Is empty: {}", stack.is_empty());

    // Works with strings too!
    let mut string_stack = Stack::new();
    string_stack.push(String::from("Hello"));
    string_stack.push(String::from("World"));

    while let Some(word) = string_stack.pop() {
        println!("{}", word);
    }
}
```

**Output:**
```
Stack size: 3
Top item: Some(3)
Popped: 3
Popped: 2
Popped: 1
Is empty: true
World
Hello
```

### **Challenge: Generic Queue**

```rust
struct Queue<T> {
    items: Vec<T>,
}

impl<T> Queue<T> {
    fn new() -> Self {
        Queue { items: Vec::new() }
    }

    fn enqueue(&mut self, item: T) {
        self.items.push(item);
    }

    fn dequeue(&mut self) -> Option<T> {
        if self.items.is_empty() {
            None
        } else {
            Some(self.items.remove(0))
        }
    }

    fn peek(&self) -> Option<&T> {
        self.items.first()
    }

    fn is_empty(&self) -> bool {
        self.items.is_empty()
    }

    fn size(&self) -> usize {
        self.items.len()
    }

    fn clear(&mut self) {
        self.items.clear();
    }
}

impl<T: std::fmt::Display> Queue<T> {
    fn display(&self) {
        print!("Queue [front -> back]: ");
        for item in &self.items {
            print!("{} ", item);
        }
        println!();
    }
}

fn main() {
    let mut queue = Queue::new();

    println!("=== Integer Queue ===");
    queue.enqueue(10);
    queue.enqueue(20);
    queue.enqueue(30);
    queue.enqueue(40);

    queue.display();
    println!("Size: {}", queue.size());

    if let Some(front) = queue.peek() {
        println!("Front: {}", front);
    }

    println!("\nDequeuing:");
    while let Some(item) = queue.dequeue() {
        println!("  Removed: {}", item);
    }

    println!("Is empty: {}", queue.is_empty());

    println!("\n=== String Queue ===");
    let mut task_queue = Queue::new();
    task_queue.enqueue("Process payment");
    task_queue.enqueue("Send email");
    task_queue.enqueue("Update database");

    task_queue.display();

    if let Some(next_task) = task_queue.dequeue() {
        println!("Processing: {}", next_task);
    }

    task_queue.display();
}
```

**Output:**
```
=== Integer Queue ===
Queue [front -> back]: 10 20 30 40
Size: 4
Front: 10

Dequeuing:
  Removed: 10
  Removed: 20
  Removed: 30
  Removed: 40
Is empty: true

=== String Queue ===
Queue [front -> back]: Process payment Send email Update database
Processing: Process payment
Queue [front -> back]: Send email Update database
```

## Generic Type Aliases

```rust
type Result<T> = std::result::Result<T, String>;

fn divide(a: i32, b: i32) -> Result<i32> {
    if b == 0 {
        Err(String::from("Division by zero"))
    } else {
        Ok(a / b)
    }
}
```

## Performance: Zero-Cost Abstraction

Generics have **no runtime cost**! Rust uses **monomorphization**:

```rust
fn process<T>(value: T) {
    // Generic code
}

fn main() {
    process(42);        // Compiler generates process_i32()
    process(3.14);      // Compiler generates process_f64()
    process("hello");   // Compiler generates process_str()
}
```

The compiler creates specialized versions for each type used, resulting in the same performance as writing separate functions!

## Common Patterns

### **Pattern 1: Builder with Generics**

```rust
struct Builder<T> {
    value: T,
}

impl<T> Builder<T> {
    fn new(value: T) -> Self {
        Builder { value }
    }

    fn transform<U, F>(self, f: F) -> Builder<U>
    where
        F: Fn(T) -> U,
    {
        Builder { value: f(self.value) }
    }

    fn build(self) -> T {
        self.value
    }
}

fn main() {
    let result = Builder::new(5)
        .transform(|x| x * 2)
        .transform(|x| x + 10)
        .build();

    println!("Result: {}", result);  // 20
}
```

### **Pattern 2: Generic Wrapper**

```rust
struct Wrapper<T> {
    value: T,
}

impl<T: std::fmt::Display> Wrapper<T> {
    fn print(&self) {
        println!("Wrapped: {}", self.value);
    }
}
```

## Key Takeaways

- ✅ Generics eliminate code duplication
- ✅ Use `<T>` to declare type parameters
- ✅ Generics work with functions, structs, enums, and methods
- ✅ Trait bounds constrain what types can be used
- ✅ Multiple type parameters: `<T, U>`
- ✅ `where` clauses improve readability
- ✅ Generics have zero runtime cost (monomorphization)
- ✅ `Option<T>` and `Result<T, E>` are generic enums
- ✅ Generic code is type-safe and checked at compile time
- ✅ Combine generics with traits for powerful abstractions

**Next**: Common traits that make Rust types work seamlessly!

---

**Progress**: Module 8, Lesson 2 complete (45/60 lessons total)
