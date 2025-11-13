# Module 2, Lesson 3: Repeating While True — The `while` Loop

## The Concept: Loop with a Built-In Condition

Imagine washing dishes. You think: "WHILE there are dirty dishes, keep washing."

You're repeating an action (washing) as long as a condition is true (dishes remain).

The `while` loop is like `loop`, but with a built-in condition check. It's cleaner than using `loop` with `if` and `break`.

## The `while` Loop

**Syntax:**

```rust
while condition {
    // Code runs while condition is true
}
```

**Example:**

```rust
fn main() {
    let mut count = 1;

    while count <= 5 {
        println!("Count: {}", count);
        count += 1;
    }

    println!("Done!");
}
```

Output:
```
Count: 1
Count: 2
Count: 3
Count: 4
Count: 5
Done!
```

The loop continues WHILE `count <= 5` is true.

## Comparing `loop` vs. `while`

**Using `loop`:**

```rust
fn main() {
    let mut count = 1;

    loop {
        if count > 5 {
            break;
        }

        println!("Count: {}", count);
        count += 1;
    }
}
```

**Using `while` (cleaner):**

```rust
fn main() {
    let mut count = 1;

    while count <= 5 {
        println!("Count: {}", count);
        count += 1;
    }
}
```

Same result, but `while` is more readable!

## Common `while` Loop Patterns

**Countdown:**

```rust
fn main() {
    let mut countdown = 10;

    while countdown > 0 {
        println!("{}...", countdown);
        countdown -= 1;
    }

    println!("Liftoff!");
}
```

**Waiting for a condition:**

```rust
fn main() {
    let mut temperature = 20;

    while temperature < 100 {
        println!("Temperature: {}°C (heating...)", temperature);
        temperature += 10;  // Simulate heating
    }

    println!("Water is boiling!");
}
```

**Processing until empty:**

```rust
fn main() {
    let mut remaining_tasks = 5;

    while remaining_tasks > 0 {
        println!("Completing task... {} left", remaining_tasks);
        remaining_tasks -= 1;
    }

    println!("All tasks done!");
}
```

## Using `break` and `continue` in `while`

You can still use `break` and `continue`:

```rust
fn main() {
    let mut num = 0;

    while num < 10 {
        num += 1;

        if num % 2 == 0 {
            continue;  // Skip even numbers
        }

        if num == 7 {
            break;  // Exit early
        }

        println!("{}", num);
    }
}
```

Output:
```
1
3
5
```

## Infinite `while` Loop (Avoid!)

This creates an infinite loop (like `loop`):

```rust
while true {
    println!("Forever!");
}
```

**Better to use `loop`** if you want an infinite loop:

```rust
loop {
    println!("Forever!");
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new while_practice
cd while_practice
code .
```

### **Experiment 1: Basic while Loop**

```rust
fn main() {
    let mut count = 1;

    while count <= 10 {
        println!("{}", count);
        count += 1;
    }
}
```

### **Experiment 2: Countdown Timer**

```rust
fn main() {
    let mut seconds = 5;

    while seconds > 0 {
        println!("{}...", seconds);
        seconds -= 1;
    }

    println!("Time's up!");
}
```

### **Experiment 3: Sum Until Limit**

```rust
fn main() {
    let mut sum = 0;
    let mut num = 1;

    while sum < 100 {
        sum += num;
        num += 1;
    }

    println!("Final sum: {}", sum);
    println!("Took {} numbers", num - 1);
}
```

### **Challenge: Guessing Game (Simplified)**

Create a simple number-guessing simulation:

```rust
fn main() {
    let secret_number = 7;
    let mut guess = 1;

    while guess != secret_number {
        println!("Guessing {}... wrong!", guess);
        guess += 1;
    }

    println!("Found it! The number was {}", secret_number);
}
```

**Extension**: Try skipping certain numbers with `continue`:

```rust
fn main() {
    let secret_number = 7;
    let mut guess = 0;

    while guess != secret_number {
        guess += 1;

        if guess % 2 == 0 {
            continue;  // Skip even guesses
        }

        println!("Trying {}...", guess);
    }

    println!("Success!");
}
```

## When to Use `while`

Use `while` when:
- You know the condition but not the exact number of iterations
- The condition is simple and readable
- You want cleaner code than `loop` with `break`

Use `for` (next lesson) when you know exactly how many times to iterate!

## Key Takeaways

- ✅ `while condition { }` loops while condition is true
- ✅ Cleaner than `loop` with `if/break` for simple conditions
- ✅ Common pattern: `while count < limit`
- ✅ Can use `break` and `continue`
- ✅ Condition is checked BEFORE each iteration
- ✅ If condition is initially false, loop never runs
