# Module 2, Lesson 1: Making Choices — The `if/else` Decision Maker

## The Concept: Choosing Different Paths

Imagine you're leaving your house. You look outside and think:
- "IF it's raining, I'll bring an umbrella"
- "OTHERWISE (ELSE), I'll leave the umbrella at home"

You're making a decision based on a condition. Programs need to make decisions too!

**In programming**: "IF this condition is true, do this. ELSE, do that."

## The `if` Statement

**Basic syntax:**

```rust
if condition {
    // Code runs if condition is true
}
```

**Example:**

```rust
fn main() {
    let temperature = 30;

    if temperature > 25 {
        println!("It's hot outside!");
    }
}
```

If `temperature` is greater than 25, it prints the message. Otherwise, nothing happens.

## The `if/else` Statement

To do something when the condition is FALSE:

```rust
if condition {
    // Code runs if true
} else {
    // Code runs if false
}
```

**Example:**

```rust
fn main() {
    let temperature = 20;

    if temperature > 25 {
        println!("It's hot outside!");
    } else {
        println!("It's nice outside!");
    }
}
```

## The `else if` Chain

For multiple conditions:

```rust
if condition1 {
    // Runs if condition1 is true
} else if condition2 {
    // Runs if condition2 is true
} else {
    // Runs if all conditions are false
}
```

**Example:**

```rust
fn main() {
    let temperature = 15;

    if temperature > 30 {
        println!("It's very hot!");
    } else if temperature > 20 {
        println!("It's warm!");
    } else if temperature > 10 {
        println!("It's cool!");
    } else {
        println!("It's cold!");
    }
}
```

## Comparison Operators

These create boolean (true/false) conditions:

| Operator | Meaning | Example |
|----------|---------|---------|
| `==` | Equal to | `x == 5` |
| `!=` | Not equal to | `x != 5` |
| `>` | Greater than | `x > 5` |
| `<` | Less than | `x < 5` |
| `>=` | Greater than or equal | `x >= 5` |
| `<=` | Less than or equal | `x <= 5` |

**Example:**

```rust
fn main() {
    let age = 18;

    if age >= 18 {
        println!("You can vote!");
    } else {
        println!("You're too young to vote.");
    }
}
```

## Logical Operators

Combine multiple conditions:

- `&&` — AND (both must be true)
- `||` — OR (at least one must be true)
- `!` — NOT (inverts true/false)

**Examples:**

```rust
fn main() {
    let age = 20;
    let has_license = true;

    // AND: Both conditions must be true
    if age >= 18 && has_license {
        println!("You can drive!");
    }

    // OR: At least one must be true
    if age < 18 || !has_license {
        println!("You cannot drive.");
    }
}
```

## `if` as an Expression

In Rust, `if` can return a value:

```rust
fn main() {
    let temperature = 30;

    let weather = if temperature > 25 {
        "hot"
    } else {
        "cool"
    };

    println!("The weather is {}", weather);
}
```

Both branches must return the same type!

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new if_else_practice
cd if_else_practice
code .
```

### **Experiment 1: Basic if/else**

```rust
fn main() {
    let number = 10;

    if number > 0 {
        println!("The number is positive");
    } else {
        println!("The number is zero or negative");
    }
}
```

### **Experiment 2: else if Chain**

```rust
fn main() {
    let score = 85;

    if score >= 90 {
        println!("Grade: A");
    } else if score >= 80 {
        println!("Grade: B");
    } else if score >= 70 {
        println!("Grade: C");
    } else if score >= 60 {
        println!("Grade: D");
    } else {
        println!("Grade: F");
    }
}
```

### **Experiment 3: Logical Operators**

```rust
fn main() {
    let age = 25;
    let has_ticket = true;

    if age >= 18 && has_ticket {
        println!("You can enter the concert!");
    } else {
        println!("Sorry, you cannot enter.");
    }
}
```

### **Challenge: Number Classifier**

Write a program that:
1. Takes a number
2. Prints whether it's:
   - Positive, negative, or zero
   - Even or odd (hint: use `number % 2 == 0` for even)

```rust
fn main() {
    let number = 7;

    // Check positive/negative/zero
    if number > 0 {
        println!("Positive");
    } else if number < 0 {
        println!("Negative");
    } else {
        println!("Zero");
    }

    // Check even/odd
    if number % 2 == 0 {
        println!("Even");
    } else {
        println!("Odd");
    }
}
```

## Key Takeaways

- ✅ `if` makes decisions based on conditions
- ✅ `if/else` provides an alternative path
- ✅ `else if` chains multiple conditions
- ✅ Comparison operators: `==`, `!=`, `>`, `<`, `>=`, `<=`
- ✅ Logical operators: `&&` (AND), `||` (OR), `!` (NOT)
- ✅ `if` can return a value in Rust
- ✅ Conditions must be boolean (`true` or `false`)
