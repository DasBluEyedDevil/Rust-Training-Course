# Module 5, Lesson 3: Looking Things Up by Name — Hash Maps (`HashMap<K, V>`)

## The Concept: Dictionary or Phone Book

Imagine a phone book:
- **Key**: Person's name
- **Value**: Phone number
- Look up phone number by name (fast!)

**Hash Maps** store key-value pairs for fast lookups.

## Creating Hash Maps

```rust
use std::collections::HashMap;

// Empty HashMap
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Red"), 50);

// From vectors
let teams = vec![String::from("Blue"), String::from("Red")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams
    .iter()
    .zip(initial_scores.iter())
    .collect();
```

## Basic Operations

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

// Insert
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Red"), 50);

// Get value (returns Option)
match scores.get("Blue") {
    Some(score) => println!("Blue: {}", score),
    None => println!("No score for Blue"),
}

// Check if key exists
if scores.contains_key("Blue") {
    println!("Blue team exists");
}

// Remove
scores.remove("Red");
```

## Updating Values

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

// Overwrite
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);  // Overwrites 10

// Only insert if key doesn't exist
scores.entry(String::from("Red")).or_insert(50);
scores.entry(String::from("Red")).or_insert(100);  // Doesn't change (still 50)

// Update based on old value
let text = "hello world wonderful world";
let mut word_count = HashMap::new();

for word in text.split_whitespace() {
    let count = word_count.entry(word).or_insert(0);
    *count += 1;  // Increment count
}

println!("{:?}", word_count);
// {"hello": 1, "world": 2, "wonderful": 1}
```

## Iterating

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Red"), 50);

// Iterate over key-value pairs
for (key, value) in &scores {
    println!("{}: {}", key, value);
}

// Iterate over keys
for key in scores.keys() {
    println!("Key: {}", key);
}

// Iterate over values
for value in scores.values() {
    println!("Value: {}", value);
}
```

## Ownership

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);

// field_name and field_value are moved!
// println!("{}", field_name);  // ❌ Error

// For types that implement Copy (like i32), values are copied
let mut map = HashMap::new();
let x = 5;
map.insert("number", x);
println!("{}", x);  // ✅ OK (i32 implements Copy)
```

## Common Patterns

### **Counting Occurrences**

```rust
use std::collections::HashMap;

fn count_words(text: &str) -> HashMap<&str, usize> {
    let mut counts = HashMap::new();

    for word in text.split_whitespace() {
        *counts.entry(word).or_insert(0) += 1;
    }

    counts
}

fn main() {
    let text = "the quick brown fox jumps over the lazy dog the fox";
    let counts = count_words(text);

    for (word, count) in counts {
        println!("{}: {}", word, count);
    }
}
```

### **Grouping by Category**

```rust
use std::collections::HashMap;

struct Person {
    name: String,
    age: u32,
}

fn group_by_age(people: Vec<Person>) -> HashMap<u32, Vec<String>> {
    let mut groups = HashMap::new();

    for person in people {
        groups
            .entry(person.age)
            .or_insert(Vec::new())
            .push(person.name);
    }

    groups
}
```

### **Caching/Memoization**

```rust
use std::collections::HashMap;

fn expensive_calculation(n: i32) -> i32 {
    println!("Calculating for {}...", n);
    n * 2  // Simulate expensive operation
}

fn main() {
    let mut cache: HashMap<i32, i32> = HashMap::new();

    for &num in &[1, 2, 1, 3, 2, 1] {
        let result = cache
            .entry(num)
            .or_insert_with(|| expensive_calculation(num));

        println!("Result for {}: {}", num, result);
    }
}
```

## Hands-On Practice

```bash
cargo new hashmap_practice
cd hashmap_practice
code .
```

### **Experiment 1: Basic HashMap**

```rust
use std::collections::HashMap;

fn main() {
    let mut ages = HashMap::new();

    ages.insert("Alice", 30);
    ages.insert("Bob", 25);
    ages.insert("Charlie", 35);

    if let Some(age) = ages.get("Alice") {
        println!("Alice is {} years old", age);
    }
}
```

### **Experiment 2: Word Frequency**

```rust
use std::collections::HashMap;

fn main() {
    let text = "hello world hello rust world hello";
    let mut freq = HashMap::new();

    for word in text.split_whitespace() {
        *freq.entry(word).or_insert(0) += 1;
    }

    for (word, count) in freq {
        println!("{}: {}", word, count);
    }
}
```

### **Challenge: Grade Book**

```rust
use std::collections::HashMap;

fn main() {
    let mut grades: HashMap<String, Vec<i32>> = HashMap::new();

    // Add grades
    grades
        .entry(String::from("Alice"))
        .or_insert(Vec::new())
        .push(95);

    grades
        .entry(String::from("Alice"))
        .or_insert(Vec::new())
        .push(88);

    grades
        .entry(String::from("Bob"))
        .or_insert(Vec::new())
        .extend(vec![78, 85, 92]);

    // Calculate averages
    for (name, scores) in &grades {
        let sum: i32 = scores.iter().sum();
        let avg = sum as f64 / scores.len() as f64;

        println!("{}: {:?} (avg: {:.2})", name, scores, avg);
    }
}
```

## Key Takeaways

- ✅ `HashMap<K, V>` stores key-value pairs
- ✅ Import with `use std::collections::HashMap;`
- ✅ `.insert(key, value)` adds/updates
- ✅ `.get(key)` returns `Option<&V>`
- ✅ `.entry(key).or_insert(default)` for conditional insert
- ✅ Keys must implement `Eq` and `Hash`
- ✅ Owned types are moved into HashMap
- ✅ Fast lookups (O(1) average)
- ✅ Great for counting, caching, grouping

**Next**: Practice project bringing together all collections!
