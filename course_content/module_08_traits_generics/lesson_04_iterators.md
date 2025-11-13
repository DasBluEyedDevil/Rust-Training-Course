# Module 8, Lesson 4: Processing Collections Elegantly — Iterators

## The Concept: Assembly Line

Imagine an assembly line:
- **Items flow** through stations
- **Each station** transforms or inspects items
- **No manual looping** - the line handles movement

**Iterators** let you process sequences without manual index management.

## What Are Iterators?

Iterators provide a way to process elements in a sequence one at a time:

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];

    // Old way: manual loop
    for i in 0..numbers.len() {
        println!("{}", numbers[i]);
    }

    // Iterator way: cleaner
    for num in numbers.iter() {
        println!("{}", num);
    }

    // Even simpler
    for num in &numbers {
        println!("{}", num);
    }
}
```

## Creating Iterators

### **From Collections**

```rust
fn main() {
    let vec = vec![1, 2, 3];

    let iter1 = vec.iter();          // Immutable references
    let iter2 = vec.iter_mut();      // Mutable references (requires mut vec)
    let iter3 = vec.into_iter();     // Takes ownership
}
```

### **From Ranges**

```rust
fn main() {
    for i in 0..5 {
        println!("{}", i);  // 0, 1, 2, 3, 4
    }

    for i in 0..=5 {
        println!("{}", i);  // 0, 1, 2, 3, 4, 5 (inclusive)
    }
}
```

## Iterator Methods (Adapters)

### **map() - Transform Each Element**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];

    let doubled: Vec<i32> = numbers
        .iter()
        .map(|x| x * 2)
        .collect();

    println!("{:?}", doubled);  // [2, 4, 6, 8, 10]
}
```

### **filter() - Keep Matching Elements**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let evens: Vec<&i32> = numbers
        .iter()
        .filter(|x| *x % 2 == 0)
        .collect();

    println!("{:?}", evens);  // [2, 4, 6, 8, 10]
}
```

### **Chaining Methods**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let result: Vec<i32> = numbers
        .iter()
        .filter(|x| *x % 2 == 0)  // Keep evens
        .map(|x| x * x)            // Square them
        .collect();

    println!("{:?}", result);  // [4, 16, 36, 64, 100]
}
```

## Consumer Methods

### **collect() - Build a Collection**

```rust
fn main() {
    let numbers = vec![1, 2, 3];

    // Collect into Vec
    let vec: Vec<i32> = numbers.iter().map(|x| x * 2).collect();

    // Collect into different types
    use std::collections::HashSet;
    let set: HashSet<i32> = numbers.iter().copied().collect();

    println!("Vec: {:?}", vec);
    println!("Set: {:?}", set);
}
```

### **sum() and product()**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];

    let total: i32 = numbers.iter().sum();
    let product: i32 = numbers.iter().product();

    println!("Sum: {}", total);        // 15
    println!("Product: {}", product);  // 120
}
```

### **count()**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let even_count = numbers
        .iter()
        .filter(|x| *x % 2 == 0)
        .count();

    println!("Even numbers: {}", even_count);  // 5
}
```

### **any() and all()**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];

    let has_even = numbers.iter().any(|x| x % 2 == 0);
    let all_positive = numbers.iter().all(|x| *x > 0);

    println!("Has even: {}", has_even);           // true
    println!("All positive: {}", all_positive);   // true
}
```

### **find() and position()**

```rust
fn main() {
    let numbers = vec![1, 3, 5, 7, 8, 9];

    let first_even = numbers.iter().find(|x| *x % 2 == 0);
    let position = numbers.iter().position(|x| *x % 2 == 0);

    println!("First even: {:?}", first_even);  // Some(8)
    println!("Position: {:?}", position);      // Some(4)
}
```

### **min() and max()**

```rust
fn main() {
    let numbers = vec![5, 2, 8, 1, 9, 3];

    let min = numbers.iter().min();
    let max = numbers.iter().max();

    println!("Min: {:?}", min);  // Some(1)
    println!("Max: {:?}", max);  // Some(9)
}
```

## Advanced Iterator Methods

### **take() and skip()**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let first_three: Vec<&i32> = numbers.iter().take(3).collect();
    let skip_two: Vec<&i32> = numbers.iter().skip(2).collect();

    println!("First 3: {:?}", first_three);  // [1, 2, 3]
    println!("Skip 2: {:?}", skip_two);      // [3, 4, 5, 6, 7, 8, 9, 10]
}
```

### **enumerate() - With Indices**

```rust
fn main() {
    let words = vec!["hello", "world", "rust"];

    for (i, word) in words.iter().enumerate() {
        println!("{}: {}", i, word);
    }
}
```

**Output:**
```
0: hello
1: world
2: rust
```

### **zip() - Combine Two Iterators**

```rust
fn main() {
    let names = vec!["Alice", "Bob", "Charlie"];
    let ages = vec![30, 25, 35];

    for (name, age) in names.iter().zip(ages.iter()) {
        println!("{} is {} years old", name, age);
    }
}
```

**Output:**
```
Alice is 30 years old
Bob is 25 years old
Charlie is 35 years old
```

### **fold() - Accumulate a Value**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];

    let sum = numbers.iter().fold(0, |acc, x| acc + x);
    let product = numbers.iter().fold(1, |acc, x| acc * x);

    println!("Sum: {}", sum);        // 15
    println!("Product: {}", product); // 120
}
```

### **flat_map() - Flatten Nested Structures**

```rust
fn main() {
    let nested = vec![vec![1, 2], vec![3, 4], vec![5, 6]];

    let flattened: Vec<i32> = nested
        .iter()
        .flat_map(|inner| inner.iter())
        .copied()
        .collect();

    println!("{:?}", flattened);  // [1, 2, 3, 4, 5, 6]
}
```

## Hands-On Practice

```bash
cargo new iterator_practice
cd iterator_practice
code .
```

### **Experiment 1: Data Processing Pipeline**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // Complex transformation pipeline
    let result: Vec<i32> = numbers
        .iter()
        .filter(|x| *x > 3)           // Keep > 3
        .map(|x| x * x)                // Square
        .filter(|x| *x < 50)           // Keep < 50
        .collect();

    println!("Result: {:?}", result);  // [16, 25, 36, 49]
}
```

### **Experiment 2: Text Processing**

```rust
fn main() {
    let text = "The quick brown fox jumps over the lazy dog";

    let word_lengths: Vec<usize> = text
        .split_whitespace()
        .map(|word| word.len())
        .collect();

    println!("Word lengths: {:?}", word_lengths);

    let long_words: Vec<&str> = text
        .split_whitespace()
        .filter(|word| word.len() > 3)
        .collect();

    println!("Long words: {:?}", long_words);

    let total_chars: usize = text
        .split_whitespace()
        .map(|word| word.len())
        .sum();

    println!("Total characters: {}", total_chars);
}
```

### **Experiment 3: Statistics Calculator**

```rust
fn main() {
    let scores = vec![85.5, 90.0, 78.5, 92.0, 88.5, 95.0, 82.0];

    let count = scores.len();
    let sum: f64 = scores.iter().sum();
    let average = sum / count as f64;

    let min = scores.iter().cloned().fold(f64::INFINITY, f64::min);
    let max = scores.iter().cloned().fold(f64::NEG_INFINITY, f64::max);

    let above_90 = scores.iter().filter(|x| **x >= 90.0).count();

    println!("=== Score Statistics ===");
    println!("Count: {}", count);
    println!("Sum: {:.2}", sum);
    println!("Average: {:.2}", average);
    println!("Min: {:.2}", min);
    println!("Max: {:.2}", max);
    println!("Scores >= 90: {}", above_90);
}
```

**Output:**
```
=== Score Statistics ===
Count: 7
Sum: 611.50
Average: 87.36
Min: 78.50
Max: 95.00
Scores >= 90: 3
```

### **Challenge: Log File Analyzer**

```rust
fn main() {
    let log_lines = vec![
        "[INFO] Application started",
        "[ERROR] Database connection failed",
        "[INFO] Retrying connection",
        "[WARN] High memory usage",
        "[ERROR] Authentication failed",
        "[INFO] User logged in",
        "[ERROR] File not found",
        "[INFO] Task completed",
    ];

    // Count by level
    let error_count = log_lines
        .iter()
        .filter(|line| line.contains("[ERROR]"))
        .count();

    let warn_count = log_lines
        .iter()
        .filter(|line| line.contains("[WARN]"))
        .count();

    let info_count = log_lines
        .iter()
        .filter(|line| line.contains("[INFO]"))
        .count();

    // Extract error messages
    let errors: Vec<String> = log_lines
        .iter()
        .filter(|line| line.contains("[ERROR]"))
        .map(|line| line.replace("[ERROR] ", ""))
        .collect();

    // Find first warning
    let first_warning = log_lines
        .iter()
        .find(|line| line.contains("[WARN]"));

    println!("=== Log Analysis ===");
    println!("Total lines: {}", log_lines.len());
    println!("ERROR: {}", error_count);
    println!("WARN: {}", warn_count);
    println!("INFO: {}", info_count);

    println!("\n=== Errors ===");
    for (i, error) in errors.iter().enumerate() {
        println!("{}. {}", i + 1, error);
    }

    if let Some(warning) = first_warning {
        println!("\n=== First Warning ===");
        println!("{}", warning);
    }
}
```

**Output:**
```
=== Log Analysis ===
Total lines: 8
ERROR: 3
WARN: 1
INFO: 4

=== Errors ===
1. Database connection failed
2. Authentication failed
3. File not found

=== First Warning ===
[WARN] High memory usage
```

## Creating Custom Iterators

```rust
struct Counter {
    count: u32,
    max: u32,
}

impl Counter {
    fn new(max: u32) -> Counter {
        Counter { count: 0, max }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        if self.count < self.max {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}

fn main() {
    let counter = Counter::new(5);

    for num in counter {
        println!("{}", num);  // 1, 2, 3, 4, 5
    }

    // Can use all iterator methods!
    let sum: u32 = Counter::new(10).sum();
    println!("Sum of 1-10: {}", sum);  // 55
}
```

## Performance: Iterators are Fast!

Iterators are **zero-cost abstractions** - they're as fast as hand-written loops!

```rust
// Iterator way (clean and fast)
let sum: i32 = numbers.iter().sum();

// Manual way (same performance)
let mut sum = 0;
for num in &numbers {
    sum += num;
}
```

The compiler optimizes iterator chains into efficient machine code!

## Common Patterns

### **Pattern 1: Processing Options**

```rust
fn main() {
    let numbers = vec![Some(1), None, Some(3), None, Some(5)];

    let values: Vec<i32> = numbers
        .into_iter()
        .filter_map(|x| x)  // Keep only Some values
        .collect();

    println!("{:?}", values);  // [1, 3, 5]
}
```

### **Pattern 2: Group Processing**

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let (evens, odds): (Vec<_>, Vec<_>) = numbers
        .into_iter()
        .partition(|x| x % 2 == 0);

    println!("Evens: {:?}", evens);  // [2, 4, 6, 8, 10]
    println!("Odds: {:?}", odds);    // [1, 3, 5, 7, 9]
}
```

### **Pattern 3: String Building**

```rust
fn main() {
    let words = vec!["Rust", "is", "awesome"];

    let sentence: String = words
        .iter()
        .map(|s| *s)
        .collect::<Vec<&str>>()
        .join(" ");

    println!("{}", sentence);  // Rust is awesome
}
```

## Key Takeaways

- ✅ Iterators process sequences without manual indexing
- ✅ `.iter()` for immutable, `.iter_mut()` for mutable, `.into_iter()` for ownership
- ✅ **Adapters** transform iterators: `map()`, `filter()`, `take()`, `skip()`
- ✅ **Consumers** produce final values: `collect()`, `sum()`, `count()`, `find()`
- ✅ Chain methods for powerful data pipelines
- ✅ Iterators are **zero-cost abstractions** (no performance penalty)
- ✅ Use `enumerate()` for indices, `zip()` to combine iterators
- ✅ `fold()` accumulates values, `flat_map()` flattens nested structures
- ✅ Custom iterators implement the `Iterator` trait
- ✅ Iterators make code more expressive and functional

**Next**: Practice project combining traits, generics, and iterators!

---

**Progress**: Module 8, Lesson 4 complete (47/60 lessons total)
