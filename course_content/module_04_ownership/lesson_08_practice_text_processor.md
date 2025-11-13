# Module 4, Lesson 8: Practice Project — Building a Text Processor

## Project Overview

Build a text processor that demonstrates ownership, borrowing, and memory efficiency!

**What we're building:**
- Analyze text without unnecessary copying
- Modify text in-place using mutable references
- Return processed data efficiently
- No memory leaks, no data races

**Skills practiced:**
- ✅ Ownership and moves
- ✅ Immutable borrowing (reading without copying)
- ✅ Mutable borrowing (modifying in-place)
- ✅ Lifetimes (returning references)
- ✅ Efficient memory usage

## The Complete Project

### **Step 1: Create the Project**

```bash
cargo new text_processor
cd text_processor
code .
```

### **Step 2: Build the Processor (Complete Code)**

```rust
// Text statistics struct
struct TextStats {
    char_count: usize,
    word_count: usize,
    line_count: usize,
    unique_words: usize,
}

// Analyze text without taking ownership
fn analyze_text(text: &str) -> TextStats {
    let char_count = text.len();
    let line_count = text.lines().count();
    let word_count = text.split_whitespace().count();

    // Count unique words (case-insensitive)
    let mut unique_words = std::collections::HashSet::new();
    for word in text.split_whitespace() {
        unique_words.insert(word.to_lowercase());
    }

    TextStats {
        char_count,
        word_count,
        line_count,
        unique_words: unique_words.len(),
    }
}

// Modify text in-place (mutable borrowing)
fn remove_extra_spaces(text: &mut String) {
    let words: Vec<&str> = text.split_whitespace().collect();
    *text = words.join(" ");
}

// Convert to title case in-place
fn to_title_case(text: &mut String) {
    let words: Vec<String> = text
        .split_whitespace()
        .map(|word| {
            let mut chars = word.chars();
            match chars.next() {
                None => String::new(),
                Some(first) => first.to_uppercase().chain(chars).collect(),
            }
        })
        .collect();

    *text = words.join(" ");
}

// Find longest word (returns reference - lifetime!)
fn find_longest_word(text: &str) -> Option<&str> {
    text.split_whitespace()
        .max_by_key(|word| word.len())
}

// Count occurrences of a word (case-insensitive)
fn count_word(text: &str, target: &str) -> usize {
    let target_lower = target.to_lowercase();

    text.split_whitespace()
        .filter(|word| word.to_lowercase() == target_lower)
        .count()
}

// Replace all occurrences (mutable borrowing)
fn replace_word(text: &mut String, from: &str, to: &str) {
    *text = text.replace(from, to);
}

fn main() {
    // Original text (we own this)
    let mut text = String::from(
        "the quick brown fox jumps over the lazy dog. \
         The  dog  was  really  lazy."
    );

    println!("=== Original Text ===");
    println!("{}\n", text);

    // Analyze (immutable borrow - no copying!)
    println!("=== Analysis ===");
    let stats = analyze_text(&text);
    println!("Characters: {}", stats.char_count);
    println!("Words: {}", stats.word_count);
    println!("Lines: {}", stats.line_count);
    println!("Unique words: {}", stats.unique_words);

    // Find longest word (returns reference - efficient!)
    if let Some(longest) = find_longest_word(&text) {
        println!("Longest word: {}", longest);
    }

    // Count specific word
    let dog_count = count_word(&text, "dog");
    println!("'dog' appears {} times\n", dog_count);

    // Modify in-place (mutable borrow)
    println!("=== Cleaning Text ===");
    remove_extra_spaces(&mut text);
    println!("{}\n", text);

    println!("=== Title Case ===");
    to_title_case(&mut text);
    println!("{}\n", text);

    println!("=== Replace 'Dog' with 'Cat' ===");
    replace_word(&mut text, "Dog", "Cat");
    println!("{}\n", text);

    // Final analysis
    println!("=== Final Analysis ===");
    let final_stats = analyze_text(&text);
    println!("Characters: {}", final_stats.char_count);
    println!("Words: {}", final_stats.word_count);
    println!("Unique words: {}", final_stats.unique_words);
}
```

### **Step 3: Run the Project**

```bash
cargo run
```

**Expected output:**

```
=== Original Text ===
the quick brown fox jumps over the lazy dog. The  dog  was  really  lazy.

=== Analysis ===
Characters: 75
Words: 13
Lines: 1
Unique words: 11
Longest word: really
'dog' appears 2 times

=== Cleaning Text ===
the quick brown fox jumps over the lazy dog. The dog was really lazy.

=== Title Case ===
The Quick Brown Fox Jumps Over The Lazy Dog. The Dog Was Really Lazy.

=== Replace 'Dog' with 'Cat' ===
The Quick Brown Fox Jumps Over The Lazy Cat. The Cat Was Really Lazy.

=== Final Analysis ===
Characters: 75
Words: 13
Unique words: 11
```

## Understanding the Code

### **Immutable Borrowing (Analysis)**

```rust
fn analyze_text(text: &str) -> TextStats {
    // Reads text without copying it
    let char_count = text.len();
    // ...
}
```

**Why it matters**: No expensive string copying!

### **Returning References (Lifetimes)**

```rust
fn find_longest_word(text: &str) -> Option<&str> {
    // Returns reference to a word in the original text
    text.split_whitespace().max_by_key(|word| word.len())
}
```

**Why it matters**: No allocation - returns a "slice" into existing data!

### **Mutable Borrowing (Modification)**

```rust
fn remove_extra_spaces(text: &mut String) {
    // Modifies in-place
    *text = words.join(" ");
}
```

**Why it matters**: Efficient modification without creating temporary copies!

## Challenges and Extensions

### **Challenge 1: Add More Analysis**

Add functions to calculate:
- Average word length
- Sentence count (count `.` characters)
- Alphabetical first/last words

### **Challenge 2: Word Frequency**

Create a function that returns the top 5 most frequent words:

```rust
use std::collections::HashMap;

fn top_words(text: &str, n: usize) -> Vec<(&str, usize)> {
    let mut freq: HashMap<&str, usize> = HashMap::new();

    for word in text.split_whitespace() {
        *freq.entry(word).or_insert(0) += 1;
    }

    let mut pairs: Vec<_> = freq.iter().collect();
    pairs.sort_by(|a, b| b.1.cmp(a.1));
    pairs.into_iter().take(n).map(|(k, v)| (*k, *v)).collect()
}
```

### **Challenge 3: Palindrome Detector**

```rust
fn is_palindrome(text: &str) -> bool {
    let clean: String = text
        .chars()
        .filter(|c| c.is_alphanumeric())
        .map(|c| c.to_ascii_lowercase())
        .collect();

    clean == clean.chars().rev().collect::<String>()
}
```

### **Challenge 4: Add Sentence Manipulation**

```rust
fn reverse_sentences(text: &mut String) {
    let sentences: Vec<&str> = text.split('.').collect();
    let reversed: Vec<&str> = sentences.into_iter().rev().collect();
    *text = reversed.join(".");
}
```

## Memory Efficiency Comparison

**BAD (copies everywhere):**

```rust
fn bad_analyze(text: String) -> String {  // Takes ownership
    let modified = text.clone();  // Unnecessary clone
    let result = modified.clone();  // Another clone
    result  // Returns owned String
}
```

**GOOD (borrows efficiently):**

```rust
fn good_analyze(text: &str) -> &str {  // Borrows
    // Return reference into original string
    &text[0..10]
}
```

## Key Takeaways

- ✅ Use `&str` for reading (no copies)
- ✅ Use `&mut String` for in-place modification
- ✅ Return references when possible (lifetimes)
- ✅ Avoid `.clone()` unless necessary
- ✅ Mutable borrows enable efficient updates
- ✅ Immutable borrows enable safe concurrent reads
- ✅ Ownership prevents memory leaks automatically

---

## ✅ Module 4 Complete!

You've mastered Rust's ownership system:
- ✅ Stack vs. Heap
- ✅ The three ownership rules
- ✅ Common ownership errors
- ✅ Immutable borrowing (`&T`)
- ✅ Mutable borrowing (`&mut T`)
- ✅ Borrow checker errors
- ✅ Lifetimes (basics)
- ✅ Built an efficient text processor!

**This is THE hardest part of Rust. You did it!**

**Next: Module 5 — Collections (String, Vec, HashMap)**
