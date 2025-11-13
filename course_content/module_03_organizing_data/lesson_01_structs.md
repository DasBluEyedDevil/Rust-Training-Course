# Module 3, Lesson 1: Grouping Related Information — Introduction to Structs

## The Concept: Bundling Data Together

Imagine you're creating a contact list. For each person, you need to store:
- Name
- Age
- Email
- Phone number

You could create separate variables:

```rust
let person1_name = "Alice";
let person1_age = 30;
let person1_email = "alice@example.com";
let person1_phone = "555-1234";
```

But this gets messy fast! What if you have 100 people? You need a way to group related information together.

Think of a **struct** as a custom container that holds multiple related pieces of data, like a form with labeled fields.

## What is a Struct?

A **struct** (short for "structure") is a custom data type that lets you package related values together.

**Syntax:**

```rust
struct StructName {
    field1: Type1,
    field2: Type2,
    field3: Type3,
}
```

**Example:**

```rust
struct Person {
    name: String,
    age: i32,
    email: String,
}
```

This defines a new type called `Person` with three fields.

## Creating Struct Instances

To create an actual `Person`:

```rust
fn main() {
    let alice = Person {
        name: String::from("Alice"),
        age: 30,
        email: String::from("alice@example.com"),
    };

    println!("Name: {}", alice.name);
    println!("Age: {}", alice.age);
    println!("Email: {}", alice.email);
}

struct Person {
    name: String,
    age: i32,
    email: String,
}
```

**Accessing fields:** Use dot notation: `alice.name`

## Mutable Structs

To modify fields, make the entire struct mutable:

```rust
fn main() {
    let mut alice = Person {
        name: String::from("Alice"),
        age: 30,
        email: String::from("alice@example.com"),
    };

    println!("Age: {}", alice.age);

    alice.age = 31;  // Birthday!

    println!("New age: {}", alice.age);
}

struct Person {
    name: String,
    age: i32,
    email: String,
}
```

## Functions with Structs

Pass structs to functions:

```rust
fn main() {
    let alice = Person {
        name: String::from("Alice"),
        age: 30,
        email: String::from("alice@example.com"),
    };

    print_person(&alice);  // Borrow the struct
}

fn print_person(person: &Person) {
    println!("Name: {}", person.name);
    println!("Age: {}", person.age);
    println!("Email: {}", person.email);
}

struct Person {
    name: String,
    age: i32,
    email: String,
}
```

## Methods on Structs

You can define functions that belong to a struct using `impl`:

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method that calculates area
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Method to check if it's square
    fn is_square(&self) -> bool {
        self.width == self.height
    }
}

fn main() {
    let rect = Rectangle {
        width: 10,
        height: 20,
    };

    println!("Area: {}", rect.area());
    println!("Is square: {}", rect.is_square());
}
```

**Key points:**
- `impl Rectangle` — "implement methods for Rectangle"
- `&self` — Reference to the struct instance
- Call methods with dot notation: `rect.area()`

## Constructor Functions

Create helper functions to build structs:

```rust
impl Person {
    fn new(name: String, age: i32, email: String) -> Person {
        Person { name, age, email }
    }
}

fn main() {
    let alice = Person::new(
        String::from("Alice"),
        30,
        String::from("alice@example.com"),
    );

    println!("{}", alice.name);
}

struct Person {
    name: String,
    age: i32,
    email: String,
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new structs_practice
cd structs_practice
code .
```

### **Experiment 1: Basic Struct**

```rust
struct Book {
    title: String,
    author: String,
    pages: u32,
}

fn main() {
    let my_book = Book {
        title: String::from("The Rust Book"),
        author: String::from("Steve Klabnik"),
        pages: 500,
    };

    println!("Title: {}", my_book.title);
    println!("Author: {}", my_book.author);
    println!("Pages: {}", my_book.pages);
}
```

### **Experiment 2: Methods**

```rust
struct Circle {
    radius: f64,
}

impl Circle {
    fn area(&self) -> f64 {
        3.14159 * self.radius * self.radius
    }

    fn circumference(&self) -> f64 {
        2.0 * 3.14159 * self.radius
    }
}

fn main() {
    let circle = Circle { radius: 5.0 };

    println!("Radius: {}", circle.radius);
    println!("Area: {:.2}", circle.area());
    println!("Circumference: {:.2}", circle.circumference());
}
```

### **Challenge: Create a Student Struct**

Create a `Student` struct with:
- `name` (String)
- `student_id` (u32)
- `gpa` (f64)

Add methods:
- `new()` — Constructor
- `is_honor_roll()` — Returns true if GPA >= 3.5
- `print_info()` — Prints all student information

```rust
struct Student {
    name: String,
    student_id: u32,
    gpa: f64,
}

impl Student {
    fn new(name: String, student_id: u32, gpa: f64) -> Student {
        Student { name, student_id, gpa }
    }

    fn is_honor_roll(&self) -> bool {
        self.gpa >= 3.5
    }

    fn print_info(&self) {
        println!("Name: {}", self.name);
        println!("ID: {}", self.student_id);
        println!("GPA: {:.2}", self.gpa);
        println!("Honor Roll: {}", self.is_honor_roll());
    }
}

fn main() {
    let student = Student::new(
        String::from("Bob"),
        12345,
        3.8,
    );

    student.print_info();
}
```

## Key Takeaways

- ✅ Structs group related data together
- ✅ `struct Name { field: Type }` defines a struct
- ✅ Create instances with `Name { field: value }`
- ✅ Access fields with dot notation: `instance.field`
- ✅ Entire struct must be `mut` to modify fields
- ✅ `impl` adds methods to structs
- ✅ `&self` is the first parameter in methods
- ✅ Constructor patterns: `fn new()` is common
