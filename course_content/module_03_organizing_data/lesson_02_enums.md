# Module 3, Lesson 2: Multiple Possibilities — Introduction to Enums

## The Concept: One of Several Options

Imagine a traffic light. It can only be one of three colors at any given time:
- Red
- Yellow
- Green

It can't be multiple colors simultaneously, and it can't be purple or blue—only these three specific options.

An **enum** (enumeration) represents data that can be one of several distinct possibilities.

## What is an Enum?

An **enum** defines a type by listing its possible variants.

**Syntax:**

```rust
enum EnumName {
    Variant1,
    Variant2,
    Variant3,
}
```

**Example:**

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}
```

Now `TrafficLight` is a type that can only be one of these three values.

## Using Enums

Create enum values:

```rust
fn main() {
    let light = TrafficLight::Red;  // :: accesses enum variants

    // We'll see how to use this value next!
}

enum TrafficLight {
    Red,
    Yellow,
    Green,
}
```

## Matching on Enums

Use `match` to handle each possibility:

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

fn main() {
    let light = TrafficLight::Red;

    match light {
        TrafficLight::Red => println!("Stop!"),
        TrafficLight::Yellow => println!("Slow down!"),
        TrafficLight::Green => println!("Go!"),
    }
}
```

**Important:** `match` must handle ALL possible variants. The compiler checks!

## Enums with Data

Enums can store additional data with each variant:

```rust
enum Message {
    Quit,                       // No data
    Move { x: i32, y: i32 },   // Named fields (like a struct)
    Write(String),              // Single value
    ChangeColor(u8, u8, u8),   // Three values (RGB)
}

fn main() {
    let msg1 = Message::Quit;
    let msg2 = Message::Move { x: 10, y: 20 };
    let msg3 = Message::Write(String::from("Hello!"));
    let msg4 = Message::ChangeColor(255, 0, 0);

    process_message(msg2);
}

fn process_message(msg: Message) {
    match msg {
        Message::Quit => println!("Quitting..."),
        Message::Move { x, y } => println!("Move to ({}, {})", x, y),
        Message::Write(text) => println!("Message: {}", text),
        Message::ChangeColor(r, g, b) => println!("Color: rgb({}, {}, {})", r, g, b),
    }
}
```

## Common Use Cases

**Representing states:**

```rust
enum ConnectionState {
    Connected,
    Connecting,
    Disconnected,
}

fn main() {
    let state = ConnectionState::Connecting;

    match state {
        ConnectionState::Connected => println!("You're online!"),
        ConnectionState::Connecting => println!("Connecting..."),
        ConnectionState::Disconnected => println!("Offline"),
    }
}
```

**Representing different types of data:**

```rust
enum WebEvent {
    PageLoad,
    PageUnload,
    KeyPress(char),
    Click { x: i32, y: i32 },
}

fn handle_event(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("Page loaded"),
        WebEvent::PageUnload => println!("Page unloaded"),
        WebEvent::KeyPress(c) => println!("Key pressed: {}", c),
        WebEvent::Click { x, y } => println!("Clicked at ({}, {})", x, y),
    }
}

fn main() {
    handle_event(WebEvent::PageLoad);
    handle_event(WebEvent::KeyPress('a'));
    handle_event(WebEvent::Click { x: 100, y: 200 });
}
```

## Methods on Enums

Enums can have methods too:

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

impl TrafficLight {
    fn can_go(&self) -> bool {
        match self {
            TrafficLight::Green => true,
            TrafficLight::Red | TrafficLight::Yellow => false,
        }
    }

    fn duration_seconds(&self) -> u32 {
        match self {
            TrafficLight::Red => 60,
            TrafficLight::Yellow => 5,
            TrafficLight::Green => 55,
        }
    }
}

fn main() {
    let light = TrafficLight::Green;

    println!("Can go: {}", light.can_go());
    println!("Duration: {} seconds", light.duration_seconds());
}
```

## Hands-On Practice

### **Create a Practice Project**

```bash
cargo new enums_practice
cd enums_practice
code .
```

### **Experiment 1: Basic Enum**

```rust
enum Direction {
    North,
    South,
    East,
    West,
}

fn main() {
    let dir = Direction::North;

    match dir {
        Direction::North => println!("Going north!"),
        Direction::South => println!("Going south!"),
        Direction::East => println!("Going east!"),
        Direction::West => println!("Going west!"),
    }
}
```

### **Experiment 2: Enum with Data**

```rust
enum Shape {
    Circle(f64),                    // radius
    Rectangle(f64, f64),            // width, height
    Triangle(f64, f64, f64),        // three sides
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(r) => 3.14159 * r * r,
            Shape::Rectangle(w, h) => w * h,
            Shape::Triangle(a, b, c) => {
                // Heron's formula
                let s = (a + b + c) / 2.0;
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
        }
    }
}

fn main() {
    let circle = Shape::Circle(5.0);
    let rectangle = Shape::Rectangle(10.0, 5.0);
    let triangle = Shape::Triangle(3.0, 4.0, 5.0);

    println!("Circle area: {:.2}", circle.area());
    println!("Rectangle area: {:.2}", rectangle.area());
    println!("Triangle area: {:.2}", triangle.area());
}
```

### **Challenge: Create a Coin Enum**

Create an enum representing US coins with their cent values:
- Penny (1)
- Nickel (5)
- Dime (10)
- Quarter (25)

Add a method `value_in_cents()` that returns the value.

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

impl Coin {
    fn value_in_cents(&self) -> u32 {
        match self {
            Coin::Penny => 1,
            Coin::Nickel => 5,
            Coin::Dime => 10,
            Coin::Quarter => 25,
        }
    }
}

fn main() {
    let coins = vec![
        Coin::Penny,
        Coin::Nickel,
        Coin::Dime,
        Coin::Quarter,
    ];

    let mut total = 0;
    for coin in coins {
        total += coin.value_in_cents();
    }

    println!("Total value: {} cents", total);
}
```

## Key Takeaways

- ✅ Enums represent data that can be one of several variants
- ✅ `enum Name { Variant1, Variant2 }` defines an enum
- ✅ Access variants with `Name::Variant`
- ✅ `match` handles all possible variants
- ✅ Enums can hold data: `Variant(Type)` or `Variant { field: Type }`
- ✅ Compiler ensures you handle all cases
- ✅ Enums can have methods with `impl`
- ✅ More type-safe than using multiple booleans or magic numbers
