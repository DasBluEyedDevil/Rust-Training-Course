# Module 2, Lesson 4: Counting Through a List — The `for` Loop

## The Concept: Iterating Over a Collection

Imagine you have 5 boxes numbered 1 through 5, and you need to check each one:
- "FOR each box, open it and check inside"

You're going through a known collection of items, one by one.

The `for` loop is perfect when you know exactly what you're iterating over: a range of numbers, a list, or any collection.

## The `for` Loop

**Syntax:**

```rust
for variable in collection {
    // Code runs for each item in collection
}
```

**Example with a range:**

```rust
fn main() {
    for number in 1..6 {  // 1, 2, 3, 4, 5 (excludes 6)
        println!("Number: {}", number);
    }
}
```

Output:
```
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
```

## Ranges

Rust has two types of ranges:

**Exclusive range (doesn't include the end):**

```rust
for i in 1..5 {  // 1, 2, 3, 4
    println!("{}", i);
}
```

**Inclusive range (includes the end):**

```rust
for i in 1..=5 {  // 1, 2, 3, 4, 5
    println!("{}", i);
}
```

## Comparing Loop Types

**Using `while` (manual counting):**

```rust
fn main() {
    let mut i = 1;

    while i <= 5 {
        println!("{}", i);
        i += 1;
    }
}
```

**Using `for` (cleaner and safer):**

```rust
fn main() {
    for i in 1..=5 {
        println!("{}", i);
    }
}
```

The `for` loop is more concise and prevents forgetting to increment the counter!

## Common Patterns

**Countdown:**

```rust
fn main() {
    for num in (1..=10).rev() {  // .rev() reverses the range
        println!("{}...", num);
    }
    println!("Liftoff!");
}
```

**Iterating with steps:**

```rust
fn main() {
    for num in (0..=20).step_by(2) {  // Every 2nd number
        println!("{}", num);
    }
}
```

Output: `0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20`

**Iterating over an array:**

```rust
fn main() {
    let fruits = ["apple", "banana", "cherry"];

    for fruit in fruits {
        println!("I like {}s!", fruit);
    }
}
```

Output:
```
I like apples!
I like bananas!
I like cherrys!
```

## Using `break` and `continue`

Works the same as in other loops:

```rust
fn main() {
    for num in 1..=10 {
        if num == 5 {
            continue;  // Skip 5
        }

        if num == 8 {
            break;  // Stop at 8
        }

        println!("{}", num);
    }
}
```

Output:
```
1
2
3
4
6
7
```

## Nested `for` Loops

Create multiplication tables:

```rust
fn main() {
    for i in 1..=3 {
        for j in 1..=3 {
            println!("{} x {} = {}", i, j, i * j);
        }
        println!("---");
    }
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new for_practice
cd for_practice
code .
```

### **Experiment 1: Basic Range**

```rust
fn main() {
    println!("Counting to 10:");

    for i in 1..=10 {
        println!("{}", i);
    }
}
```

### **Experiment 2: Countdown**

```rust
fn main() {
    println!("Countdown:");

    for num in (1..=5).rev() {
        println!("{}...", num);
    }

    println!("Blastoff!");
}
```

### **Experiment 3: Even Numbers**

```rust
fn main() {
    println!("Even numbers from 0 to 20:");

    for num in (0..=20).step_by(2) {
        println!("{}", num);
    }
}
```

### **Experiment 4: Multiplication Table**

```rust
fn main() {
    let number = 7;

    println!("Multiplication table for {}:", number);

    for i in 1..=10 {
        println!("{} x {} = {}", number, i, number * i);
    }
}
```

### **Challenge: FizzBuzz**

Classic programming challenge:
- Print numbers 1 to 30
- If divisible by 3: print "Fizz"
- If divisible by 5: print "Buzz"
- If divisible by both: print "FizzBuzz"
- Otherwise: print the number

```rust
fn main() {
    for num in 1..=30 {
        if num % 3 == 0 && num % 5 == 0 {
            println!("FizzBuzz");
        } else if num % 3 == 0 {
            println!("Fizz");
        } else if num % 5 == 0 {
            println!("Buzz");
        } else {
            println!("{}", num);
        }
    }
}
```

## When to Use `for`

Use `for` when:
- You know the collection or range you're iterating over
- You want to go through each item once
- You want clean, safe iteration

Use `while` when the exit condition is more complex!

## Key Takeaways

- ✅ `for variable in collection { }` iterates over items
- ✅ `1..5` is an exclusive range (1, 2, 3, 4)
- ✅ `1..=5` is an inclusive range (1, 2, 3, 4, 5)
- ✅ `.rev()` reverses a range
- ✅ `.step_by(n)` skips n items
- ✅ `for` is safer and cleaner than manual `while` counting
- ✅ Can use `break` and `continue`
- ✅ Great for arrays, ranges, and collections
