# Module 5, Lesson 4: Practice Project — Building a Word Counter

## Project Overview

Build a word frequency analyzer that demonstrates all three collection types!

**What we're building:**
- Analyze text from multiple sources
- Count word frequencies
- Find most/least common words
- Display statistics

**Skills practiced:**
- ✅ String manipulation
- ✅ Vector operations
- ✅ HashMap for counting
- ✅ All collection types working together

## The Complete Project

### **Step 1: Create the Project**

```bash
cargo new word_counter
cd word_counter
code .
```

### **Step 2: Build the Counter (Complete Code)**

```rust
use std::collections::HashMap;

// Analyze text and return word frequencies
fn count_words(text: &str) -> HashMap<String, usize> {
    let mut counts = HashMap::new();

    for word in text.split_whitespace() {
        let word = word
            .chars()
            .filter(|c| c.is_alphanumeric())
            .collect::<String>()
            .to_lowercase();

        if !word.is_empty() {
            *counts.entry(word).or_insert(0) += 1;
        }
    }

    counts
}

// Find most common words
fn top_words(counts: &HashMap<String, usize>, n: usize) -> Vec<(&String, &usize)> {
    let mut pairs: Vec<_> = counts.iter().collect();
    pairs.sort_by(|a, b| b.1.cmp(a.1));  // Sort by count (descending)
    pairs.into_iter().take(n).collect()
}

// Find least common words
fn least_common_words(counts: &HashMap<String, usize>, n: usize) -> Vec<(&String, &usize)> {
    let mut pairs: Vec<_> = counts.iter().collect();
    pairs.sort_by(|a, b| a.1.cmp(b.1));  // Sort by count (ascending)
    pairs.into_iter().take(n).collect()
}

// Calculate statistics
fn calculate_stats(counts: &HashMap<String, usize>) -> Stats {
    let total_words: usize = counts.values().sum();
    let unique_words = counts.len();

    let average_frequency = if unique_words > 0 {
        total_words as f64 / unique_words as f64
    } else {
        0.0
    };

    Stats {
        total_words,
        unique_words,
        average_frequency,
    }
}

struct Stats {
    total_words: usize,
    unique_words: usize,
    average_frequency: f64,
}

// Find words that appear exactly n times
fn words_with_count(counts: &HashMap<String, usize>, target: usize) -> Vec<String> {
    counts
        .iter()
        .filter(|(_, &count)| count == target)
        .map(|(word, _)| word.clone())
        .collect()
}

fn main() {
    let text = "The quick brown fox jumps over the lazy dog. \
                The dog was really lazy, and the fox was quick. \
                The lazy dog slept while the quick fox ran.";

    println!("=== Text Analysis ===\n");
    println!("Text: {}\n", text);

    // Count words
    let counts = count_words(text);

    // Display statistics
    let stats = calculate_stats(&counts);
    println!("=== Statistics ===");
    println!("Total words: {}", stats.total_words);
    println!("Unique words: {}", stats.unique_words);
    println!("Average frequency: {:.2}\n", stats.average_frequency);

    // Top 5 most common words
    println!("=== Top 5 Most Common Words ===");
    for (word, count) in top_words(&counts, 5) {
        println!("{}: {} times", word, count);
    }

    // Least common words
    println!("\n=== Least Common Words ===");
    for (word, count) in least_common_words(&counts, 5) {
        println!("{}: {} times", word, count);
    }

    // Words that appear exactly once
    println!("\n=== Words Appearing Once ===");
    let once = words_with_count(&counts, 1);
    println!("{:?}", once);

    // All words alphabetically
    println!("\n=== All Words (Alphabetical) ===");
    let mut words: Vec<_> = counts.keys().collect();
    words.sort();
    for word in words {
        println!("{}: {}", word, counts.get(word).unwrap());
    }
}
```

### **Step 3: Run the Project**

```bash
cargo run
```

**Expected output:**

```
=== Text Analysis ===

Text: The quick brown fox jumps over the lazy dog. The dog was really lazy...

=== Statistics ===
Total words: 25
Unique words: 11
Average frequency: 2.27

=== Top 5 Most Common Words ===
the: 5 times
lazy: 3 times
quick: 3 times
fox: 3 times
dog: 3 times

=== Least Common Words ===
over: 1 times
jumps: 1 times
brown: 1 times
slept: 1 times
while: 1 times

=== Words Appearing Once ===
["over", "jumps", "brown", "slept", "while", "really", "ran", "was"]

=== All Words (Alphabetical) ===
brown: 1
dog: 3
fox: 3
...
```

## Understanding the Code

### **String Processing**

```rust
let word = word
    .chars()
    .filter(|c| c.is_alphanumeric())  // Remove punctuation
    .collect::<String>()
    .to_lowercase();  // Normalize case
```

### **HashMap for Counting**

```rust
*counts.entry(word).or_insert(0) += 1;
```

### **Vector for Sorting**

```rust
let mut pairs: Vec<_> = counts.iter().collect();
pairs.sort_by(|a, b| b.1.cmp(a.1));  // Sort by count
```

## Challenges and Extensions

### **Challenge 1: Read from File**

```rust
use std::fs;

fn main() {
    let text = fs::read_to_string("input.txt")
        .expect("Failed to read file");

    let counts = count_words(&text);
    // ... rest of code
}
```

### **Challenge 2: Filter Common Words**

```rust
fn filter_common_words(
    counts: &HashMap<String, usize>,
    stop_words: &[&str],
) -> HashMap<String, usize> {
    counts
        .iter()
        .filter(|(word, _)| !stop_words.contains(&word.as_str()))
        .map(|(k, v)| (k.clone(), *v))
        .collect()
}

fn main() {
    let stop_words = vec!["the", "a", "an", "and", "or", "but"];
    let filtered = filter_common_words(&counts, &stop_words);
}
```

### **Challenge 3: Word Length Analysis**

```rust
fn average_word_length(counts: &HashMap<String, usize>) -> f64 {
    let total_length: usize = counts
        .keys()
        .map(|word| word.len())
        .sum();

    total_length as f64 / counts.len() as f64
}

fn words_longer_than(counts: &HashMap<String, usize>, min_len: usize) -> Vec<String> {
    counts
        .keys()
        .filter(|word| word.len() > min_len)
        .cloned()
        .collect()
}
```

### **Challenge 4: Save Results to File**

```rust
use std::fs::File;
use std::io::Write;

fn save_results(counts: &HashMap<String, usize>, filename: &str) -> std::io::Result<()> {
    let mut file = File::create(filename)?;

    let mut pairs: Vec<_> = counts.iter().collect();
    pairs.sort_by(|a, b| b.1.cmp(a.1));

    for (word, count) in pairs {
        writeln!(file, "{},{}", word, count)?;
    }

    Ok(())
}
```

## Key Takeaways

- ✅ Strings for text processing
- ✅ HashMaps for counting frequencies
- ✅ Vectors for sorting results
- ✅ All three collections working together
- ✅ Iterator methods for filtering and mapping
- ✅ Real-world text analysis application
- ✅ Efficient data processing without unnecessary copying

---

## ✅ Module 5 Complete!

You've mastered Rust collections:
- ✅ String vs &str (owned vs borrowed text)
- ✅ Vec<T> (growable lists)
- ✅ HashMap<K, V> (key-value storage)
- ✅ Built a complete word frequency analyzer!

**Next: Module 6 — Advanced Error Handling**

Total Progress: 33 lessons complete (~55%)
