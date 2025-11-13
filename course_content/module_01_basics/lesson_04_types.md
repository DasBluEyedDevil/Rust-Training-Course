# Module 1, Lesson 4: Numbers and Text — The Basic Types of Data

## Understanding Data Types

Imagine organizing a filing cabinet with different types of information:
- **Whole numbers**: 1, 2, 3, 100 (counting apples)
- **Decimal numbers**: 3.14, 98.6, -5.5 (temperature, measurements)
- **Yes/No answers**: true or false (Is it raining?)
- **Text**: Words and sentences (names, messages)

You wouldn't mix these up—you can't do math with words, or spell sentences with numbers. Each type of information has its own rules.

Computers need to know what KIND of data they're working with. This is called the data's **type**.

## Rust's Basic Types

**Integers (whole numbers):**
- `i32` — Signed integer (can be negative), default for whole numbers
  - Range: about -2 billion to +2 billion
- `i64` — Larger signed integer
- `u32` — Unsigned (positive only)

**Floating-point (decimal numbers):**
- `f64` — 64-bit decimal number, default for decimals
  - Examples: 3.14, -0.5, 100.0

**Boolean (true/false):**
- `bool` — Either `true` or `false`

**Characters:**
- `char` — Single character (use single quotes)
  - Examples: `'a'`, `'Z'`, `'😀'`

**Strings (text):**
- `&str` — Text in double quotes: `"Hello"`
- `String` — Owned, growable text (Module 5)

## Type Inference

Rust can usually figure out types automatically:

```rust
let age = 30;  // Rust knows this is i32
let pi = 3.14; // Rust knows this is f64
```

But you can specify explicitly:

```rust
let age: i32 = 30;
let pi: f64 = 3.14;
```

## Code Examples

**Creating variables of different types:**

```rust
fn main() {
    // Integers
    let apples = 5;
    let big_number: i64 = 1_000_000;  // Underscores for readability

    // Floating-point
    let pi = 3.14;
    let temperature: f32 = 98.6;

    // Boolean
    let is_raining = false;
    let is_sunny = true;

    // Characters (single quotes!)
    let letter = 'A';
    let emoji = '😀';

    // Strings (double quotes!)
    let name = "Alice";
    let greeting = "Hello!";

    println!("Apples: {}", apples);
    println!("Pi: {}", pi);
    println!("Is raining: {}", is_raining);
    println!("Letter: {}", letter);
    println!("Name: {}", name);
}
```

**Math operations:**

```rust
fn main() {
    let x = 10;
    let y = 3;

    let sum = x + y;          // 13
    let difference = x - y;   // 7
    let product = x * y;      // 30
    let quotient = x / y;     // 3 (integer division)
    let remainder = x % y;    // 1 (modulo)

    println!("Sum: {}", sum);
    println!("Product: {}", product);
    println!("Remainder: {}", remainder);
}
```

**Type mismatch (ERROR):**

```rust
fn main() {
    let integer_num = 10;
    let float_num = 3.5;

    let result = integer_num + float_num;  // ❌ ERROR: mismatched types
}
```

You can't directly mix integers and floats. Rust enforces type safety!

## Hands-On Practice

### **Create a Test Project**

```bash
cargo new types_practice
cd types_practice
code .
```

### **Experiment 1: Different Types**

```rust
fn main() {
    let age = 25;
    let price = 9.99;
    let is_student = true;
    let grade = 'A';
    let name = "Bob";

    println!("Age: {}", age);
    println!("Price: ${}", price);
    println!("Is student: {}", is_student);
    println!("Grade: {}", grade);
    println!("Name: {}", name);
}
```

### **Experiment 2: Math Operations**

```rust
fn main() {
    let apples = 10;
    let people = 3;

    let apples_per_person = apples / people;
    let leftover = apples % people;

    println!("Each person gets {} apples", apples_per_person);
    println!("{} apples left over", leftover);
}
```

### **Experiment 3: Type Mismatch**

```rust
fn main() {
    let whole = 10;
    let decimal = 3.5;

    let result = whole + decimal;  // ❌ Try this and see the error!
}
```

Run `cargo run` to see the compiler error. Then comment it out.

### **Challenge: Temperature Converter**

Convert Celsius to Fahrenheit using: `F = C * 1.8 + 32.0`

```rust
fn main() {
    let celsius = 25.0;
    let fahrenheit = celsius * 1.8 + 32.0;

    println!("{}°C is {}°F", celsius, fahrenheit);
}
```

Expected output: `25°C is 77°F`

## Key Takeaways

- ✅ **Type** defines what kind of data a variable holds
- ✅ `i32` — Default integer type
- ✅ `f64` — Default floating-point type
- ✅ `bool` — True or false
- ✅ `char` — Single character (single quotes)
- ✅ `&str` — Text strings (double quotes)
- ✅ Rust infers types but you can specify them explicitly
- ✅ You can't mix types directly (type safety!)
- ✅ Math operators: `+`, `-`, `*`, `/`, `%`
