# Module 8, Lesson 5: Practice Project — Building a Generic Data Filter

## Project Overview

Build a powerful, generic data filtering and transformation system that demonstrates traits, generics, and iterators!

**What we're building:**
- Generic filter system for any data type
- Chainable transformations
- Statistical analysis
- Custom iterators
- Complete type safety

**Skills practiced:**
- ✅ Defining and implementing traits
- ✅ Generic types and functions
- ✅ Trait bounds
- ✅ Iterator methods and chains
- ✅ All advanced concepts working together

## The Complete Project

### **Step 1: Create the Project**

```bash
cargo new data_filter
cd data_filter
code .
```

### **Step 2: Build the Filter System**

```rust
use std::fmt::Display;

// ===== Filterable Trait =====

trait Filterable {
    fn matches(&self, criteria: &str) -> bool;
}

// ===== Analyzable Trait =====

trait Analyzable: Display {
    fn score(&self) -> f64;
    fn category(&self) -> String;
}

// ===== Generic Filter =====

struct DataFilter<T> {
    items: Vec<T>,
}

impl<T> DataFilter<T> {
    fn new() -> Self {
        DataFilter { items: Vec::new() }
    }

    fn add(&mut self, item: T) {
        self.items.push(item);
    }

    fn count(&self) -> usize {
        self.items.len()
    }

    fn is_empty(&self) -> bool {
        self.items.is_empty()
    }
}

impl<T: Filterable> DataFilter<T> {
    fn filter_by(&self, criteria: &str) -> Vec<&T> {
        self.items
            .iter()
            .filter(|item| item.matches(criteria))
            .collect()
    }

    fn count_matching(&self, criteria: &str) -> usize {
        self.items
            .iter()
            .filter(|item| item.matches(criteria))
            .count()
    }
}

impl<T: Analyzable> DataFilter<T> {
    fn average_score(&self) -> f64 {
        if self.items.is_empty() {
            return 0.0;
        }

        let sum: f64 = self.items.iter().map(|item| item.score()).sum();
        sum / self.items.len() as f64
    }

    fn top_scores(&self, n: usize) -> Vec<&T> {
        let mut sorted: Vec<&T> = self.items.iter().collect();
        sorted.sort_by(|a, b| {
            b.score()
                .partial_cmp(&a.score())
                .unwrap_or(std::cmp::Ordering::Equal)
        });

        sorted.into_iter().take(n).collect()
    }

    fn group_by_category(&self) -> std::collections::HashMap<String, Vec<&T>> {
        let mut groups = std::collections::HashMap::new();

        for item in &self.items {
            let category = item.category();
            groups.entry(category).or_insert_with(Vec::new).push(item);
        }

        groups
    }

    fn display_all(&self) {
        for (i, item) in self.items.iter().enumerate() {
            println!("{}. {}", i + 1, item);
        }
    }
}

// ===== Student Example =====

#[derive(Debug, Clone)]
struct Student {
    name: String,
    grade: f64,
    major: String,
}

impl Display for Student {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{} ({}) - {:.1}", self.name, self.major, self.grade)
    }
}

impl Filterable for Student {
    fn matches(&self, criteria: &str) -> bool {
        self.name.to_lowercase().contains(&criteria.to_lowercase())
            || self.major.to_lowercase().contains(&criteria.to_lowercase())
    }
}

impl Analyzable for Student {
    fn score(&self) -> f64 {
        self.grade
    }

    fn category(&self) -> String {
        self.major.clone()
    }
}

// ===== Product Example =====

#[derive(Debug, Clone)]
struct Product {
    name: String,
    price: f64,
    category: String,
    rating: f64,
}

impl Display for Product {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(
            f,
            "{} (${:.2}) - {} stars",
            self.name, self.price, self.rating
        )
    }
}

impl Filterable for Product {
    fn matches(&self, criteria: &str) -> bool {
        self.name.to_lowercase().contains(&criteria.to_lowercase())
            || self.category.to_lowercase().contains(&criteria.to_lowercase())
    }
}

impl Analyzable for Product {
    fn score(&self) -> f64 {
        self.rating
    }

    fn category(&self) -> String {
        self.category.clone()
    }
}

// ===== Generic Statistics =====

struct Statistics<T> {
    data: Vec<T>,
}

impl<T: Analyzable + Clone> Statistics<T> {
    fn new(data: Vec<T>) -> Self {
        Statistics { data }
    }

    fn summary(&self) {
        if self.data.is_empty() {
            println!("No data");
            return;
        }

        let scores: Vec<f64> = self.data.iter().map(|item| item.score()).collect();

        let sum: f64 = scores.iter().sum();
        let avg = sum / scores.len() as f64;
        let min = scores
            .iter()
            .cloned()
            .fold(f64::INFINITY, f64::min);
        let max = scores
            .iter()
            .cloned()
            .fold(f64::NEG_INFINITY, f64::max);

        println!("╔═══════════════════════════════════╗");
        println!("║      Statistical Summary          ║");
        println!("╠═══════════════════════════════════╣");
        println!("║ Count:   {:22} ║", self.data.len());
        println!("║ Average: {:22.2} ║", avg);
        println!("║ Min:     {:22.2} ║", min);
        println!("║ Max:     {:22.2} ║", max);
        println!("╚═══════════════════════════════════╝");
    }

    fn percentile(&self, p: f64) -> Option<f64> {
        if self.data.is_empty() || p < 0.0 || p > 100.0 {
            return None;
        }

        let mut scores: Vec<f64> = self.data.iter().map(|item| item.score()).collect();
        scores.sort_by(|a, b| a.partial_cmp(b).unwrap());

        let index = ((p / 100.0) * (scores.len() - 1) as f64).round() as usize;
        Some(scores[index])
    }
}

// ===== Main Program =====

fn main() {
    println!("=== Student Data Filter Demo ===\n");

    let mut students = DataFilter::new();

    students.add(Student {
        name: String::from("Alice Johnson"),
        grade: 92.5,
        major: String::from("Computer Science"),
    });

    students.add(Student {
        name: String::from("Bob Smith"),
        grade: 85.0,
        major: String::from("Mathematics"),
    });

    students.add(Student {
        name: String::from("Charlie Davis"),
        grade: 88.5,
        major: String::from("Computer Science"),
    });

    students.add(Student {
        name: String::from("Diana Wilson"),
        grade: 95.0,
        major: String::from("Physics"),
    });

    students.add(Student {
        name: String::from("Eve Brown"),
        grade: 90.5,
        major: String::from("Mathematics"),
    });

    println!("All students:");
    students.display_all();

    println!("\n--- Filtering by 'Computer' ---");
    let cs_students = students.filter_by("Computer");
    for student in cs_students {
        println!("  - {}", student);
    }

    println!("\n--- Top 3 Students ---");
    let top_3 = students.top_scores(3);
    for student in top_3 {
        println!("  - {}", student);
    }

    println!("\n--- Students by Major ---");
    let grouped = students.group_by_category();
    for (major, students_in_major) in grouped {
        println!("{}:", major);
        for student in students_in_major {
            println!("  - {}", student);
        }
    }

    println!("\n--- Statistics ---");
    println!("Average grade: {:.2}", students.average_score());
    println!("CS students: {}", students.count_matching("Computer"));

    let stats = Statistics::new(
        students
            .items
            .iter()
            .cloned()
            .collect(),
    );
    stats.summary();

    if let Some(p90) = stats.percentile(90.0) {
        println!("90th percentile: {:.2}", p90);
    }

    println!("\n\n=== Product Data Filter Demo ===\n");

    let mut products = DataFilter::new();

    products.add(Product {
        name: String::from("Laptop"),
        price: 999.99,
        category: String::from("Electronics"),
        rating: 4.5,
    });

    products.add(Product {
        name: String::from("Mouse"),
        price: 29.99,
        category: String::from("Electronics"),
        rating: 4.2,
    });

    products.add(Product {
        name: String::from("Desk"),
        price: 299.99,
        category: String::from("Furniture"),
        rating: 4.7,
    });

    products.add(Product {
        name: String::from("Chair"),
        price: 199.99,
        category: String::from("Furniture"),
        rating: 4.8,
    });

    println!("All products:");
    products.display_all();

    println!("\n--- Electronics ---");
    let electronics = products.filter_by("Electronics");
    for product in electronics {
        println!("  - {}", product);
    }

    println!("\n--- Top 2 Rated Products ---");
    let top_rated = products.top_scores(2);
    for product in top_rated {
        println!("  - {}", product);
    }

    println!("\n--- Average Rating ---");
    println!("{:.2} stars", products.average_score());
}
```

### **Step 3: Run the Project**

```bash
cargo run
```

**Expected Output:**

```
=== Student Data Filter Demo ===

All students:
1. Alice Johnson (Computer Science) - 92.5
2. Bob Smith (Mathematics) - 85.0
3. Charlie Davis (Computer Science) - 88.5
4. Diana Wilson (Physics) - 95.0
5. Eve Brown (Mathematics) - 90.5

--- Filtering by 'Computer' ---
  - Alice Johnson (Computer Science) - 92.5
  - Charlie Davis (Computer Science) - 88.5

--- Top 3 Students ---
  - Diana Wilson (Physics) - 95.0
  - Alice Johnson (Computer Science) - 92.5
  - Eve Brown (Mathematics) - 90.5

--- Students by Major ---
Mathematics:
  - Bob Smith (Mathematics) - 85.0
  - Eve Brown (Mathematics) - 90.5
Physics:
  - Diana Wilson (Physics) - 95.0
Computer Science:
  - Alice Johnson (Computer Science) - 92.5
  - Charlie Davis (Computer Science) - 88.5

--- Statistics ---
Average grade: 90.30
CS students: 2
╔═══════════════════════════════════╗
║      Statistical Summary          ║
╠═══════════════════════════════════╣
║ Count:                        5   ║
║ Average:                  90.30   ║
║ Min:                      85.00   ║
║ Max:                      95.00   ║
╚═══════════════════════════════════╝
90th percentile: 92.50


=== Product Data Filter Demo ===

All products:
1. Laptop ($999.99) - 4.5 stars
2. Mouse ($29.99) - 4.2 stars
3. Desk ($299.99) - 4.7 stars
4. Chair ($199.99) - 4.8 stars

--- Electronics ---
  - Laptop ($999.99) - 4.5 stars
  - Mouse ($29.99) - 4.2 stars

--- Top 2 Rated Products ---
  - Chair ($199.99) - 4.8 stars
  - Desk ($299.99) - 4.7 stars

--- Average Rating ---
4.55 stars
```

## Understanding the Code

### **Generic with Trait Bounds**

```rust
impl<T: Filterable> DataFilter<T> {
    fn filter_by(&self, criteria: &str) -> Vec<&T> {
        // Only available when T implements Filterable
    }
}
```

**Why**: Different functionality based on what traits T implements.

### **Multiple Trait Bounds**

```rust
trait Analyzable: Display {
    // Analyzable requires Display
}
```

**Why**: Ensures types can be both analyzed and displayed.

### **Iterator Chains**

```rust
self.items
    .iter()
    .filter(|item| item.matches(criteria))
    .collect()
```

**Why**: Clean, functional data processing.

### **Generic Statistics**

```rust
struct Statistics<T> {
    data: Vec<T>,
}

impl<T: Analyzable + Clone> Statistics<T> {
    // Works with any Analyzable type
}
```

**Why**: Single implementation works for Students, Products, and any future types!

## Challenges and Extensions

### **Challenge 1: Add Sorting Options**

```rust
impl<T: Analyzable + Clone> DataFilter<T> {
    fn sorted_by_score(&self, ascending: bool) -> Vec<T> {
        let mut sorted = self.items.clone();
        sorted.sort_by(|a, b| {
            let cmp = a.score().partial_cmp(&b.score()).unwrap();
            if ascending {
                cmp
            } else {
                cmp.reverse()
            }
        });
        sorted
    }
}
```

### **Challenge 2: Add Filtering by Range**

```rust
impl<T: Analyzable> DataFilter<T> {
    fn filter_by_score_range(&self, min: f64, max: f64) -> Vec<&T> {
        self.items
            .iter()
            .filter(|item| {
                let score = item.score();
                score >= min && score <= max
            })
            .collect()
    }
}
```

### **Challenge 3: Add Export Functionality**

```rust
impl<T: Analyzable> DataFilter<T> {
    fn export_csv(&self, filename: &str) -> std::io::Result<()> {
        use std::fs::File;
        use std::io::Write;

        let mut file = File::create(filename)?;

        writeln!(file, "Item,Score,Category")?;

        for item in &self.items {
            writeln!(
                file,
                "\"{}\",{},\"{}\"",
                item,
                item.score(),
                item.category()
            )?;
        }

        Ok(())
    }
}
```

### **Challenge 4: Custom Iterator**

```rust
struct FilterIterator<'a, T> {
    items: &'a [T],
    criteria: String,
    index: usize,
}

impl<'a, T: Filterable> Iterator for FilterIterator<'a, T> {
    type Item = &'a T;

    fn next(&mut self) -> Option<Self::Item> {
        while self.index < self.items.len() {
            let item = &self.items[self.index];
            self.index += 1;

            if item.matches(&self.criteria) {
                return Some(item);
            }
        }
        None
    }
}

impl<T: Filterable> DataFilter<T> {
    fn iter_matching(&self, criteria: &str) -> FilterIterator<T> {
        FilterIterator {
            items: &self.items,
            criteria: criteria.to_string(),
            index: 0,
        }
    }
}
```

## Key Concepts Demonstrated

### **1. Trait-Based Polymorphism**
Different types (Student, Product) work with the same generic code through shared traits.

### **2. Generic Type Parameters**
`DataFilter<T>` works with any type, providing type-safe collections.

### **3. Trait Bounds**
`impl<T: Analyzable>` ensures T has required methods at compile time.

### **4. Iterator Methods**
Powerful data processing with `filter()`, `map()`, `collect()`, `fold()`.

### **5. Zero-Cost Abstractions**
All generics compiled to efficient specialized code - no runtime overhead!

## Key Takeaways

- ✅ Traits define shared behavior across different types
- ✅ Generics enable code reuse without duplication
- ✅ Trait bounds constrain generic types at compile time
- ✅ Iterators provide expressive data processing
- ✅ Multiple trait implementations unlock different functionality
- ✅ Generic code is type-safe and performant
- ✅ Combining traits + generics + iterators creates powerful abstractions
- ✅ Real-world applications benefit from these patterns
- ✅ Single implementation works for unlimited types
- ✅ Rust's type system ensures correctness

---

## ✅ Module 8 Complete!

You've mastered advanced Rust concepts:
- ✅ Traits for shared behavior
- ✅ Generics for flexible, reusable code
- ✅ Common traits (Debug, Display, Clone, etc.)
- ✅ Iterators for elegant data processing
- ✅ Built a complete generic data filtering system!

**Next: Module 9 — Capstone Project (Task Manager Application)**

Total Progress: 48 lessons complete (80%)

---

[← Back to Module 8](README.md) | [Continue to Module 9 →](../module_09_capstone/)
