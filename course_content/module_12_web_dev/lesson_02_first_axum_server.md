# Module 12: Web Development with Rust

# Lesson 2: Your First Axum Web Server

## What is Axum?

**Axum** is a modern, ergonomic web framework for Rust built on:
- **tokio** - async runtime
- **hyper** - HTTP library
- **tower** - middleware and services

**Why Axum?**
- Fast and efficient
- Type-safe extractors
- Excellent error messages
- Built by the Tokio team
- Latest version: **0.8.0** (January 2025)

## Project Setup

### Create a New Project

```bash
cargo new web_server
cd web_server
```

### Add Dependencies

Edit `Cargo.toml`:

```toml
[package]
name = "web_server"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.8.x"
tokio = { version = "1.x", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

**Dependencies explained:**
- `axum` - Web framework
- `tokio` - Async runtime (enables `async`/`await`)
- `serde` - Serialization/deserialization
- `serde_json` - JSON support

Install dependencies:

```bash
cargo build
```

## Your First "Hello World" Server

### Step 1: Basic Server

Replace `src/main.rs`:

```rust
use axum::{
    routing::get,
    Router,
};

#[tokio::main]
async fn main() {
    // Create a router with a single route
    let app = Router::new()
        .route("/", get(hello_world));

    // Create listener
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 Server running on http://localhost:3000");

    // Run the server
    axum::serve(listener, app)
        .await
        .unwrap();
}

// Handler function
async fn hello_world() -> &'static str {
    "Hello, World!"
}
```

### Run Your Server

```bash
cargo run
```

You should see:
```
🚀 Server running on http://localhost:3000
```

### Test It

Open your browser to `http://localhost:3000` or use curl:

```bash
curl http://localhost:3000
# Output: Hello, World!
```

**Congratulations!** You've built your first web server! 🎉

## Understanding the Code

### The `#[tokio::main]` Attribute

```rust
#[tokio::main]
async fn main() {
    // async code here
}
```

**What it does:**
- Transforms `async fn main` into a regular `fn main`
- Sets up the tokio async runtime
- Allows you to use `await` in main

**Without the macro:**
```rust
// You'd have to write this manually:
fn main() {
    tokio::runtime::Runtime::new()
        .unwrap()
        .block_on(async {
            // async code here
        });
}
```

### The Router

```rust
let app = Router::new()
    .route("/", get(hello_world));
```

**Router** is the core of axum. It:
- Maps URLs (paths) to handler functions
- Supports different HTTP methods (GET, POST, PUT, DELETE)
- Can be nested and composed

### Handler Functions

```rust
async fn hello_world() -> &'static str {
    "Hello, World!"
}
```

**Handler functions:**
- Are `async` (run concurrently)
- Take extractors as parameters (more on this later)
- Return something that implements `IntoResponse`

### The Listener & Server

```rust
let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
    .await
    .unwrap();

axum::serve(listener, app)
    .await
    .unwrap();
```

**What this does:**
- Binds to port 3000 on all interfaces (`0.0.0.0`)
- Starts the server
- Waits for incoming HTTP requests

**Address explained:**
- `0.0.0.0` = accept connections from anywhere
- `127.0.0.1` or `localhost` = only local connections
- `3000` = port number (common for development)

## Multiple Routes

Let's add more routes:

```rust
use axum::{
    routing::get,
    Router,
};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(home))
        .route("/hello", get(hello))
        .route("/about", get(about));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 Server running on http://localhost:3000");

    axum::serve(listener, app).await.unwrap();
}

async fn home() -> &'static str {
    "Welcome to the home page!"
}

async fn hello() -> &'static str {
    "Hello from /hello!"
}

async fn about() -> &'static str {
    "This is a Rust web server built with axum"
}
```

**Test the routes:**

```bash
curl http://localhost:3000/
curl http://localhost:3000/hello
curl http://localhost:3000/about
```

## Returning Different Types

Handlers can return various types:

### String

```rust
async fn greeting() -> String {
    format!("Hello at {}", chrono::Local::now())
}
```

### HTML

```rust
use axum::response::Html;

async fn html_page() -> Html<&'static str> {
    Html("<h1>Hello from HTML!</h1>")
}
```

Test: Visit `http://localhost:3000/html` in your browser!

### Status Codes

```rust
use axum::http::StatusCode;

async fn not_found() -> StatusCode {
    StatusCode::NOT_FOUND
}

// Or with a message:
async fn not_found_with_message() -> (StatusCode, &'static str) {
    (StatusCode::NOT_FOUND, "Page not found")
}
```

## Complete Example with Multiple Response Types

```rust
use axum::{
    routing::get,
    Router,
    response::{Html, IntoResponse},
    http::StatusCode,
};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(home))
        .route("/html", get(html_page))
        .route("/greet", get(greeting))
        .route("/status", get(status_example))
        .route("/error", get(error_example));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 Server running on http://localhost:3000");
    println!("   Try:");
    println!("   - http://localhost:3000/");
    println!("   - http://localhost:3000/html");
    println!("   - http://localhost:3000/greet");
    println!("   - http://localhost:3000/status");
    println!("   - http://localhost:3000/error");

    axum::serve(listener, app).await.unwrap();
}

async fn home() -> &'static str {
    "Welcome! Try the different routes."
}

async fn html_page() -> Html<&'static str> {
    Html(r#"
        <!DOCTYPE html>
        <html>
            <head><title>Axum Server</title></head>
            <body>
                <h1>Hello from Axum!</h1>
                <p>This is an HTML response.</p>
            </body>
        </html>
    "#)
}

async fn greeting() -> String {
    let now = std::time::SystemTime::now()
        .duration_since(std::time::UNIX_EPOCH)
        .unwrap()
        .as_secs();

    format!("Hello! Current timestamp: {}", now)
}

async fn status_example() -> impl IntoResponse {
    (StatusCode::CREATED, "Resource created!")
}

async fn error_example() -> impl IntoResponse {
    (StatusCode::INTERNAL_SERVER_ERROR, "Oops! Something went wrong")
}
```

## Understanding `impl IntoResponse`

```rust
async fn handler() -> impl IntoResponse {
    "Can return various types"
}
```

**`IntoResponse` trait** is implemented by:
- `&str`, `String`
- `Html<T>`
- `StatusCode`
- `(StatusCode, T)` where T implements IntoResponse
- `Json<T>` (next lesson!)
- And many more...

**Why use it?**
- Flexibility - return different types from the same function
- Type safety - compiler checks compatibility

## Different HTTP Methods

```rust
use axum::routing::{get, post, put, delete};

let app = Router::new()
    .route("/users", get(list_users).post(create_user))
    .route("/users/:id", get(get_user).put(update_user).delete(delete_user));
```

**Each method can have its own handler:**

```rust
async fn list_users() -> &'static str {
    "GET /users - List all users"
}

async fn create_user() -> &'static str {
    "POST /users - Create a user"
}

async fn get_user() -> &'static str {
    "GET /users/:id - Get one user"
}

async fn update_user() -> &'static str {
    "PUT /users/:id - Update user"
}

async fn delete_user() -> &'static str {
    "DELETE /users/:id - Delete user"
}
```

## Practice Exercises

### Exercise 1: Personal Portfolio API

Create a web server with these routes:

- `GET /` → Welcome message
- `GET /bio` → Your bio (as text)
- `GET /skills` → List of your skills
- `GET /projects` → Your projects
- `GET /contact` → Contact info

**Starter code:**

```rust
use axum::{routing::get, Router};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(home))
        // Add more routes here
        ;

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 Portfolio server running!");
    axum::serve(listener, app).await.unwrap();
}

async fn home() -> &'static str {
    "Welcome to my portfolio!"
}

// Add handler functions here
```

<details>
<summary>Solution</summary>

```rust
use axum::{routing::get, Router, response::Html};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(home))
        .route("/bio", get(bio))
        .route("/skills", get(skills))
        .route("/projects", get(projects))
        .route("/contact", get(contact));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 Portfolio server running on http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}

async fn home() -> Html<&'static str> {
    Html("<h1>Welcome to my portfolio!</h1>")
}

async fn bio() -> &'static str {
    "I'm a Rust developer passionate about systems programming and web development."
}

async fn skills() -> &'static str {
    "Skills: Rust, Axum, Tokio, System Programming, Web Development"
}

async fn projects() -> &'static str {
    "Projects:\n1. TaskMaster CLI\n2. File Validator\n3. Web Server"
}

async fn contact() -> &'static str {
    "Contact: email@example.com | GitHub: github.com/username"
}
```
</details>

### Exercise 2: Status Code Practice

Create endpoints that return different status codes:

- `GET /health` → 200 OK with "Healthy"
- `GET /teapot` → 418 I'm a teapot (yes, it's real!)
- `GET /forbidden` → 403 Forbidden
- `GET /unavailable` → 503 Service Unavailable

<details>
<summary>Solution</summary>

```rust
use axum::{routing::get, Router, http::StatusCode};

async fn health() -> (StatusCode, &'static str) {
    (StatusCode::OK, "Healthy")
}

async fn teapot() -> (StatusCode, &'static str) {
    (StatusCode::IM_A_TEAPOT, "I'm a teapot ☕")
}

async fn forbidden() -> (StatusCode, &'static str) {
    (StatusCode::FORBIDDEN, "Access forbidden")
}

async fn unavailable() -> (StatusCode, &'static str) {
    (StatusCode::SERVICE_UNAVAILABLE, "Service temporarily unavailable")
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/health", get(health))
        .route("/teapot", get(teapot))
        .route("/forbidden", get(forbidden))
        .route("/unavailable", get(unavailable));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```
</details>

## Common Errors & Solutions

### Error: "error[E0277]: the trait bound `fn() -> impl Future<Output = ...>: Handler<_, _>` is not satisfied"

**Cause**: Forgot `async` on handler function

❌ **Wrong:**
```rust
fn handler() -> &'static str {  // Missing async!
    "Hello"
}
```

✅ **Correct:**
```rust
async fn handler() -> &'static str {
    "Hello"
}
```

### Error: "Address already in use"

**Cause**: Port 3000 is already being used

**Solution:**
- Stop the old server (Ctrl+C)
- Or use a different port: `"0.0.0.0:8080"`

### Error: `await` can only be used in async functions

**Cause**: Forgot `#[tokio::main]`

✅ **Correct:**
```rust
#[tokio::main]
async fn main() {
    // can use .await here
}
```

## Key Takeaways

- ✅ Axum is a modern, type-safe web framework
- ✅ `#[tokio::main]` enables async in main function
- ✅ Router maps paths to handler functions
- ✅ Handlers are async functions that return `IntoResponse`
- ✅ Different routes can use different HTTP methods
- ✅ Can return text, HTML, status codes, and more

**Next**: Routing, path parameters, and extractors!

---

**Progress**: Module 12, Lesson 2 complete (59/90+ lessons total)
