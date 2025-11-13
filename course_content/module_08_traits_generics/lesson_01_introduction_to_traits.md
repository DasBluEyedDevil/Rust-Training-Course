# Module 8, Lesson 1: Shared Behavior — Introduction to Traits

## The Concept: Contracts and Interfaces

Imagine different devices with a "chargeable" interface:
- **Phone**: Can charge (via USB-C)
- **Laptop**: Can charge (via USB-C)
- **Earbuds**: Can charge (via USB-C)

All share the same behavior: `charge()` method, but implement it differently.

**Traits** define shared behavior that types can implement.

## What Are Traits?

Traits are like interfaces in other languages - they define a set of methods that a type must implement.

```rust
// Define a trait
trait Speak {
    fn speak(&self) -> String;
}

// Implement the trait for Dog
struct Dog {
    name: String,
}

impl Speak for Dog {
    fn speak(&self) -> String {
        format!("{} says: Woof!", self.name)
    }
}

// Implement the trait for Cat
struct Cat {
    name: String,
}

impl Speak for Cat {
    fn speak(&self) -> String {
        format!("{} says: Meow!", self.name)
    }
}

fn main() {
    let dog = Dog {
        name: String::from("Buddy"),
    };

    let cat = Cat {
        name: String::from("Whiskers"),
    };

    println!("{}", dog.speak());  // Buddy says: Woof!
    println!("{}", cat.speak());  // Whiskers says: Meow!
}
```

## Why Traits Matter

### **1. Polymorphism**

Different types can be used interchangeably if they implement the same trait:

```rust
trait Drawable {
    fn draw(&self);
}

struct Circle {
    radius: f64,
}

struct Rectangle {
    width: f64,
    height: f64,
}

impl Drawable for Circle {
    fn draw(&self) {
        println!("Drawing circle with radius {}", self.radius);
    }
}

impl Drawable for Rectangle {
    fn draw(&self) {
        println!("Drawing rectangle {}x{}", self.width, self.height);
    }
}

fn render(shape: &impl Drawable) {
    shape.draw();
}

fn main() {
    let circle = Circle { radius: 5.0 };
    let rectangle = Rectangle {
        width: 10.0,
        height: 20.0,
    };

    render(&circle);     // Drawing circle with radius 5
    render(&rectangle);  // Drawing rectangle 10x20
}
```

### **2. Code Reuse**

Write functions that work with any type implementing a trait:

```rust
trait Summarizable {
    fn summary(&self) -> String;
}

struct Article {
    title: String,
    author: String,
}

struct Tweet {
    username: String,
    content: String,
}

impl Summarizable for Article {
    fn summary(&self) -> String {
        format!("'{}' by {}", self.title, self.author)
    }
}

impl Summarizable for Tweet {
    fn summary(&self) -> String {
        format!("@{}: {}", self.username, self.content)
    }
}

fn print_summary(item: &impl Summarizable) {
    println!("Summary: {}", item.summary());
}

fn main() {
    let article = Article {
        title: String::from("Rust Traits Explained"),
        author: String::from("Alice"),
    };

    let tweet = Tweet {
        username: String::from("rustlang"),
        content: String::from("Rust 1.88 released!"),
    };

    print_summary(&article);  // Summary: 'Rust Traits Explained' by Alice
    print_summary(&tweet);    // Summary: @rustlang: Rust 1.88 released!
}
```

## Default Implementations

Traits can provide default method implementations:

```rust
trait Greet {
    fn greet(&self) -> String {
        String::from("Hello!")  // Default implementation
    }

    fn farewell(&self) -> String {
        String::from("Goodbye!")
    }
}

struct Person {
    name: String,
}

struct Robot;

impl Greet for Person {
    // Override the default
    fn greet(&self) -> String {
        format!("Hello, I'm {}!", self.name)
    }
    // Use default for farewell
}

impl Greet for Robot {
    // Use both defaults (empty impl block)
}

fn main() {
    let person = Person {
        name: String::from("Alice"),
    };
    let robot = Robot;

    println!("{}", person.greet());     // Hello, I'm Alice!
    println!("{}", person.farewell());  // Goodbye!

    println!("{}", robot.greet());      // Hello!
    println!("{}", robot.farewell());   // Goodbye!
}
```

## Traits Can Have Multiple Methods

```rust
trait Shape {
    fn area(&self) -> f64;
    fn perimeter(&self) -> f64;

    fn description(&self) -> String {
        format!(
            "Shape with area {} and perimeter {}",
            self.area(),
            self.perimeter()
        )
    }
}

struct Square {
    side: f64,
}

impl Shape for Square {
    fn area(&self) -> f64 {
        self.side * self.side
    }

    fn perimeter(&self) -> f64 {
        4.0 * self.side
    }
}

fn main() {
    let square = Square { side: 5.0 };

    println!("Area: {}", square.area());
    println!("Perimeter: {}", square.perimeter());
    println!("{}", square.description());
}
```

**Output:**
```
Area: 25
Perimeter: 20
Shape with area 25 and perimeter 20
```

## Hands-On Practice

```bash
cargo new trait_practice
cd trait_practice
code .
```

### **Experiment 1: Basic Trait**

```rust
trait Identifiable {
    fn get_id(&self) -> u32;
    fn get_name(&self) -> String;
}

struct User {
    id: u32,
    username: String,
}

struct Product {
    id: u32,
    name: String,
}

impl Identifiable for User {
    fn get_id(&self) -> u32 {
        self.id
    }

    fn get_name(&self) -> String {
        self.username.clone()
    }
}

impl Identifiable for Product {
    fn get_id(&self) -> u32 {
        self.id
    }

    fn get_name(&self) -> String {
        self.name.clone()
    }
}

fn display_item(item: &impl Identifiable) {
    println!("ID: {}, Name: {}", item.get_id(), item.get_name());
}

fn main() {
    let user = User {
        id: 1,
        username: String::from("alice"),
    };

    let product = Product {
        id: 101,
        name: String::from("Laptop"),
    };

    display_item(&user);
    display_item(&product);
}
```

### **Experiment 2: Trait with Default Method**

```rust
trait Logger {
    fn log(&self, message: &str) {
        println!("[LOG] {}", message);
    }

    fn error(&self, message: &str) {
        println!("[ERROR] {}", message);
    }
}

struct Application;

struct Database;

impl Logger for Application {
    fn log(&self, message: &str) {
        println!("[APP] {}", message);  // Custom implementation
    }
}

impl Logger for Database {
    // Uses default implementations
}

fn main() {
    let app = Application;
    let db = Database;

    app.log("Application started");
    app.error("Connection failed");

    db.log("Query executed");
    db.error("Deadlock detected");
}
```

### **Experiment 3: Multiple Traits**

```rust
trait Playable {
    fn play(&self);
}

trait Pausable {
    fn pause(&self);
}

struct MusicPlayer {
    song: String,
}

impl Playable for MusicPlayer {
    fn play(&self) {
        println!("♪ Playing: {}", self.song);
    }
}

impl Pausable for MusicPlayer {
    fn pause(&self) {
        println!("⏸ Paused: {}", self.song);
    }
}

fn main() {
    let player = MusicPlayer {
        song: String::from("Rust Theme Song"),
    };

    player.play();
    player.pause();
}
```

### **Challenge: Notification System**

```rust
trait Notifiable {
    fn send(&self, message: &str);

    fn format_message(&self, message: &str) -> String {
        format!("[Notification] {}", message)
    }
}

struct EmailNotifier {
    address: String,
}

struct SMSNotifier {
    phone: String,
}

struct PushNotifier {
    device_id: String,
}

impl Notifiable for EmailNotifier {
    fn send(&self, message: &str) {
        println!("📧 Sending email to {}", self.address);
        println!("   {}", self.format_message(message));
    }

    fn format_message(&self, message: &str) -> String {
        format!("Email: {}", message)  // Custom format
    }
}

impl Notifiable for SMSNotifier {
    fn send(&self, message: &str) {
        println!("📱 Sending SMS to {}", self.phone);
        println!("   {}", self.format_message(message));
    }
}

impl Notifiable for PushNotifier {
    fn send(&self, message: &str) {
        println!("🔔 Sending push to device {}", self.device_id);
        println!("   {}", self.format_message(message));
    }
}

fn notify_all(notifiers: Vec<&dyn Notifiable>, message: &str) {
    for notifier in notifiers {
        notifier.send(message);
        println!();
    }
}

fn main() {
    let email = EmailNotifier {
        address: String::from("user@example.com"),
    };

    let sms = SMSNotifier {
        phone: String::from("+1234567890"),
    };

    let push = PushNotifier {
        device_id: String::from("device-abc-123"),
    };

    let notifiers: Vec<&dyn Notifiable> = vec![&email, &sms, &push];

    notify_all(notifiers, "Your order has been shipped!");
}
```

**Output:**
```
📧 Sending email to user@example.com
   Email: Your order has been shipped!

📱 Sending SMS to +1234567890
   [Notification] Your order has been shipped!

🔔 Sending push to device device-abc-123
   [Notification] Your order has been shipped!
```

## Trait Syntax Variants

### **impl Trait (Simple)**

```rust
fn summarize(item: &impl Summarizable) -> String {
    item.summary()
}
```

### **Trait Bound (More Flexible)**

```rust
fn summarize<T: Summarizable>(item: &T) -> String {
    item.summary()
}
```

### **Multiple Traits**

```rust
fn process(item: &(impl Readable + Writable)) {
    // item implements both traits
}
```

### **Trait Objects (Dynamic Dispatch)**

```rust
fn notify(item: &dyn Notifiable) {
    item.send("Message");
}
```

## Common Built-In Traits (Preview)

We'll explore these in detail in Lesson 3:

- **Debug**: `println!("{:?}", value)`
- **Display**: `println!("{}", value)`
- **Clone**: `value.clone()`
- **Copy**: Automatic copying
- **PartialEq**: `a == b`
- **Ord**: `a < b`, sorting

## Key Takeaways

- ✅ Traits define shared behavior across types
- ✅ Use `trait` keyword to define a trait
- ✅ Use `impl TraitName for Type` to implement traits
- ✅ Traits enable polymorphism in Rust
- ✅ Default implementations provide common behavior
- ✅ Types can implement multiple traits
- ✅ Use `&impl Trait` for function parameters
- ✅ Traits make code reusable and extensible
- ✅ `&dyn Trait` enables runtime polymorphism
- ✅ Traits are checked at compile time for safety

**Next**: Generic types for writing flexible, reusable code!

---

**Progress**: Module 8, Lesson 1 complete (44/60 lessons total)
