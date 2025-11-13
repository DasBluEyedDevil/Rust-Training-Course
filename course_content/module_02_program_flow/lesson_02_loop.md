# Module 2, Lesson 2: Repeating Forever — The `loop`

## The Concept: Doing Something Repeatedly

Imagine you're stirring soup. The recipe says: "Keep stirring until the soup starts boiling, then stop."

You're repeating an action (stirring) until a specific condition is met (boiling).

Programs often need to repeat actions. Rust has several ways to create loops. The simplest is `loop`—it repeats forever until you explicitly tell it to stop.

## The `loop` Keyword

**Basic syntax:**

```rust
loop {
    // This code repeats forever
    // ...until you 'break' out of it
}
```

**Example (infinite loop - DON'T RUN THIS!):**

```rust
fn main() {
    loop {
        println!("This prints forever!");
    }
}
```

This will print forever—you'd have to force-quit the program!

## Breaking Out of a Loop

Use `break` to exit a loop:

```rust
fn main() {
    let mut count = 0;

    loop {
        count = count + 1;
        println!("Count: {}", count);

        if count == 5 {
            break;  // Exit the loop
        }
    }

    println!("Loop finished!");
}
```

Output:
```
Count: 1
Count: 2
Count: 3
Count: 4
Count: 5
Loop finished!
```

## Skipping with `continue`

Use `continue` to skip the rest of the current iteration:

```rust
fn main() {
    let mut count = 0;

    loop {
        count = count + 1;

        if count % 2 == 0 {
            continue;  // Skip even numbers
        }

        println!("{}", count);

        if count >= 9 {
            break;
        }
    }
}
```

Output:
```
1
3
5
7
9
```

(Only odd numbers print)

## Returning Values from Loops

Loops can return values with `break`:

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;  // Return a value when breaking
        }
    };

    println!("Result: {}", result);
}
```

Output:
```
Result: 20
```

## Nested Loops

Loops inside loops:

```rust
fn main() {
    let mut outer = 0;

    loop {
        outer += 1;
        let mut inner = 0;

        loop {
            inner += 1;
            println!("outer: {}, inner: {}", outer, inner);

            if inner >= 3 {
                break;
            }
        }

        if outer >= 2 {
            break;
        }
    }
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new loop_practice
cd loop_practice
code .
```

### **Experiment 1: Countdown**

```rust
fn main() {
    let mut count = 5;

    loop {
        println!("{}", count);
        count -= 1;

        if count == 0 {
            println!("Liftoff!");
            break;
        }
    }
}
```

### **Experiment 2: Find First Multiple of 13**

```rust
fn main() {
    let mut num = 1;

    loop {
        if num % 13 == 0 {
            println!("First multiple of 13: {}", num);
            break;
        }
        num += 1;
    }
}
```

### **Experiment 3: Loop with continue**

```rust
fn main() {
    let mut count = 0;

    loop {
        count += 1;

        if count % 3 == 0 {
            continue;  // Skip multiples of 3
        }

        println!("{}", count);

        if count >= 10 {
            break;
        }
    }
}
```

### **Challenge: Sum Until Limit**

Write a program that:
1. Starts with `sum = 0` and `num = 1`
2. Keeps adding `num` to `sum`
3. Increments `num` each time
4. Stops when `sum` exceeds 100
5. Prints the final sum

```rust
fn main() {
    let mut sum = 0;
    let mut num = 1;

    loop {
        sum += num;

        if sum > 100 {
            break;
        }

        num += 1;
    }

    println!("Final sum: {}", sum);
}
```

## When to Use `loop`

Use `loop` when:
- You don't know how many iterations you need
- You want an infinite loop with `break` conditions
- The exit condition is complex

For counting a specific number of times, `for` loops (Lesson 4) are better!

## Key Takeaways

- ✅ `loop` creates an infinite loop
- ✅ `break` exits the loop
- ✅ `continue` skips to the next iteration
- ✅ Loops can return values with `break value`
- ✅ Always ensure your loop has a `break` condition!
- ✅ Use `loop` when iterations are unknown
