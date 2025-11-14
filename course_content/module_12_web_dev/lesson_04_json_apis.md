# Module 12: Web Development with Rust

# Lesson 4: JSON APIs & Request Bodies

## Working with JSON

Most modern APIs use **JSON** (JavaScript Object Notation) to send and receive data.

### The `Json` Extractor

Axum provides `Json<T>` for easy JSON handling:

```rust
use axum::{
    routing::{get, post},
    Router,
    Json,
};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
    name: String,
    email: String,
    age: u32,
}

async fn create_user(Json(user): Json<User>) -> Json<User> {
    // Automatically deserializes JSON from request body
    // Automatically serializes response to JSON
    Json(user)
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/users", post(create_user));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```

**Test with curl:**

```bash
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@example.com","age":30}'

# Response:
# {"name":"Alice","email":"alice@example.com","age":30}
```

## Complete CRUD API Example

Let's build a complete REST API for managing tasks:

```rust
use axum::{
    routing::{get, post, put, delete},
    Router,
    Json,
    extract::Path,
    http::StatusCode,
};
use serde::{Deserialize, Serialize};
use std::sync::{Arc, Mutex};

// Data model
#[derive(Serialize, Deserialize, Clone)]
struct Task {
    id: u32,
    title: String,
    completed: bool,
}

// Request types
#[derive(Deserialize)]
struct CreateTask {
    title: String,
}

#[derive(Deserialize)]
struct UpdateTask {
    title: Option<String>,
    completed: Option<bool>,
}

// Response types
#[derive(Serialize)]
struct TaskResponse {
    task: Task,
}

#[derive(Serialize)]
struct TasksResponse {
    tasks: Vec<Task>,
}

// Shared state (we'll improve this with a database later)
type AppState = Arc<Mutex<Vec<Task>>>;

> **⚠️ IMPORTANT NOTE: Temporary Learning Pattern**
>
> The `Arc<Mutex<Vec<Task>>>` pattern you see here is a **temporary tool for in-memory learning**. It is:
> - ❌ **Difficult to understand** (complex ownership and locking concepts)
> - ❌ **Performance bottleneck** (locks block other requests)
> - ❌ **Not production-ready** (data lost on restart, no persistence)
> - ❌ **Limited scalability** (single-server only, doesn't work with multiple instances)
>
> **Don't worry if this feels confusing!** In Module 13, you'll learn the **production-ready solution** using a database (PostgreSQL + SQLx), which is:
> - ✅ **Much simpler** (no Arc, no Mutex, no manual locking)
> - ✅ **Much more powerful** (complex queries, transactions, relationships)
> - ✅ **Production-ready** (persistent data, works with multiple servers)
>
> This `Arc<Mutex<T>>` pattern is just a stepping stone to help you understand state management in web servers before introducing databases. Focus on understanding the API patterns (routes, handlers, JSON) rather than mastering Arc/Mutex!

#[tokio::main]
async fn main() {
    // Initialize state
    let state: AppState = Arc::new(Mutex::new(vec![
        Task { id: 1, title: "Learn Rust".to_string(), completed: false },
        Task { id: 2, title: "Build API".to_string(), completed: false },
    ]));

    let app = Router::new()
        .route("/tasks", get(list_tasks).post(create_task))
        .route("/tasks/{id}", get(get_task).put(update_task).delete(delete_task))
        .with_state(state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 Task API running on http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}

// GET /tasks - List all tasks
async fn list_tasks(
    axum::extract::State(state): axum::extract::State<AppState>,
) -> Json<TasksResponse> {
    let tasks = state.lock().unwrap().clone();
    Json(TasksResponse { tasks })
}

// GET /tasks/:id - Get single task
async fn get_task(
    Path(id): Path<u32>,
    axum::extract::State(state): axum::extract::State<AppState>,
) -> Result<Json<TaskResponse>, StatusCode> {
    let tasks = state.lock().unwrap();
    let task = tasks.iter()
        .find(|t| t.id == id)
        .cloned();

    match task {
        Some(task) => Ok(Json(TaskResponse { task })),
        None => Err(StatusCode::NOT_FOUND),
    }
}

// POST /tasks - Create task
async fn create_task(
    axum::extract::State(state): axum::extract::State<AppState>,
    Json(create): Json<CreateTask>,
) -> (StatusCode, Json<TaskResponse>) {
    let mut tasks = state.lock().unwrap();

    let id = tasks.iter().map(|t| t.id).max().unwrap_or(0) + 1;

    let task = Task {
        id,
        title: create.title,
        completed: false,
    };

    tasks.push(task.clone());

    (StatusCode::CREATED, Json(TaskResponse { task }))
}

// PUT /tasks/:id - Update task
async fn update_task(
    Path(id): Path<u32>,
    axum::extract::State(state): axum::extract::State<AppState>,
    Json(update): Json<UpdateTask>,
) -> Result<Json<TaskResponse>, StatusCode> {
    let mut tasks = state.lock().unwrap();

    let task = tasks.iter_mut()
        .find(|t| t.id == id);

    match task {
        Some(task) => {
            if let Some(title) = update.title {
                task.title = title;
            }
            if let Some(completed) = update.completed {
                task.completed = completed;
            }
            Ok(Json(TaskResponse { task: task.clone() }))
        }
        None => Err(StatusCode::NOT_FOUND),
    }
}

// DELETE /tasks/:id - Delete task
async fn delete_task(
    Path(id): Path<u32>,
    axum::extract::State(state): axum::extract::State<AppState>,
) -> StatusCode {
    let mut tasks = state.lock().unwrap();

    let before_len = tasks.len();
    tasks.retain(|t| t.id != id);
    let after_len = tasks.len();

    if before_len > after_len {
        StatusCode::NO_CONTENT
    } else {
        StatusCode::NOT_FOUND
    }
}
```

### Test the API

```bash
# List all tasks
curl http://localhost:3000/tasks

# Get single task
curl http://localhost:3000/tasks/1

# Create task
curl -X POST http://localhost:3000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Write documentation"}'

# Update task
curl -X PUT http://localhost:3000/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"completed":true}'

# Delete task
curl -X DELETE http://localhost:3000/tasks/1
```

## Understanding Shared State

### The `Arc<Mutex<T>>` Pattern

```rust
type AppState = Arc<Mutex<Vec<Task>>>;
```

**Breaking it down:**

- **`Vec<Task>`** - The actual data (list of tasks)
- **`Mutex<T>`** - Ensures only one thread can access at a time
- **`Arc<T>`** - Allows sharing across threads (Atomic Reference Counted)

**Why needed?**
- Web servers handle multiple requests concurrently
- Each request runs in its own task/thread
- Shared data must be protected from race conditions

### Accessing State in Handlers

```rust
async fn handler(
    axum::extract::State(state): axum::extract::State<AppState>,
) -> String {
    let data = state.lock().unwrap();
    // use data
    format!("Data: {:?}", data)
}
```

**Important**: Always drop the lock quickly to avoid blocking other requests!

✅ **Good:**
```rust
async fn handler(State(state): State<AppState>) -> String {
    let result = {
        let data = state.lock().unwrap();
        data.clone() // or extract what you need
    }; // lock is dropped here

    // do other work
    format!("{:?}", result)
}
```

❌ **Bad:**
```rust
async fn handler(State(state): State<AppState>) -> String {
    let data = state.lock().unwrap();
    // long computation while holding lock - blocks other requests!
    tokio::time::sleep(Duration::from_secs(10)).await;
    format!("{:?}", data)
}
```

## Error Handling in APIs

### Custom Error Responses

```rust
use axum::{
    response::{IntoResponse, Response},
    http::StatusCode,
    Json,
};
use serde::Serialize;

#[derive(Serialize)]
struct ErrorResponse {
    error: String,
    message: String,
}

enum ApiError {
    NotFound,
    BadRequest(String),
    InternalError,
}

impl IntoResponse for ApiError {
    fn into_response(self) -> Response {
        let (status, error, message) = match self {
            ApiError::NotFound => (
                StatusCode::NOT_FOUND,
                "NOT_FOUND",
                "Resource not found",
            ),
            ApiError::BadRequest(msg) => (
                StatusCode::BAD_REQUEST,
                "BAD_REQUEST",
                &msg,
            ),
            ApiError::InternalError => (
                StatusCode::INTERNAL_SERVER_ERROR,
                "INTERNAL_ERROR",
                "Something went wrong",
            ),
        };

        let body = Json(ErrorResponse {
            error: error.to_string(),
            message: message.to_string(),
        });

        (status, body).into_response()
    }
}

// Use in handlers
async fn get_task(Path(id): Path<u32>) -> Result<Json<Task>, ApiError> {
    if id > 100 {
        return Err(ApiError::NotFound);
    }

    // ... fetch task

    Ok(Json(task))
}
```

### Validation

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct CreateUser {
    #[serde(deserialize_with = "validate_email")]
    email: String,
    #[serde(deserialize_with = "validate_age")]
    age: u32,
}

fn validate_email<'de, D>(deserializer: D) -> Result<String, D::Error>
where
    D: serde::Deserializer<'de>,
{
    let email = String::deserialize(deserializer)?;
    if email.contains('@') {
        Ok(email)
    } else {
        Err(serde::de::Error::custom("Invalid email"))
    }
}

fn validate_age<'de, D>(deserializer: D) -> Result<u32, D::Error>
where
    D: serde::Deserializer<'de>,
{
    let age = u32::deserialize(deserializer)?;
    if age >= 18 && age <= 120 {
        Ok(age)
    } else {
        Err(serde::de::Error::custom("Age must be 18-120"))
    }
}

async fn create_user(Json(user): Json<CreateUser>) -> StatusCode {
    // If we get here, validation passed
    StatusCode::CREATED
}
```

## Response Types

### Different Response Formats

```rust
use axum::response::{Html, IntoResponse};
use axum::http::header;

// JSON
async fn json_response() -> Json<User> {
    Json(User { name: "Alice".into(), email: "alice@example.com".into(), age: 30 })
}

// HTML
async fn html_response() -> Html<String> {
    Html("<h1>Hello</h1>".to_string())
}

// Plain text with custom headers
async fn custom_response() -> impl IntoResponse {
    (
        StatusCode::OK,
        [(header::CONTENT_TYPE, "text/plain")],
        "Custom response"
    )
}

// Redirect
async fn redirect() -> impl IntoResponse {
    axum::response::Redirect::to("/new-url")
}
```

### Conditional Responses

```rust
async fn get_user(Path(id): Path<u32>) -> impl IntoResponse {
    if id == 0 {
        return (StatusCode::BAD_REQUEST, "Invalid ID").into_response();
    }

    if id > 1000 {
        return (StatusCode::NOT_FOUND, "User not found").into_response();
    }

    Json(User {
        id,
        name: format!("User {}", id),
        email: format!("user{}@example.com", id),
    }).into_response()
}
```

## Practice Exercises

### Exercise 1: Book Library API

Create a REST API for managing books:

**Requirements:**
- Book model: `id`, `title`, `author`, `isbn`, `available`
- `GET /books` - list all books
- `GET /books/:id` - get single book
- `POST /books` - create book (returns 201)
- `PUT /books/:id` - update book
- `DELETE /books/:id` - delete book (returns 204)
- Use shared state with Arc<Mutex<Vec<Book>>>

<details>
<summary>Solution</summary>

```rust
use axum::{
    routing::{get, post, put, delete},
    Router,
    Json,
    extract::{Path, State},
    http::StatusCode,
};
use serde::{Deserialize, Serialize};
use std::sync::{Arc, Mutex};

#[derive(Serialize, Deserialize, Clone)]
struct Book {
    id: u32,
    title: String,
    author: String,
    isbn: String,
    available: bool,
}

#[derive(Deserialize)]
struct CreateBook {
    title: String,
    author: String,
    isbn: String,
}

#[derive(Deserialize)]
struct UpdateBook {
    title: Option<String>,
    author: Option<String>,
    available: Option<bool>,
}

type AppState = Arc<Mutex<Vec<Book>>>;

#[tokio::main]
async fn main() {
    let state: AppState = Arc::new(Mutex::new(vec![]));

    let app = Router::new()
        .route("/books", get(list_books).post(create_book))
        .route("/books/{id}", get(get_book).put(update_book).delete(delete_book))
        .with_state(state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("📚 Book Library API running!");
    axum::serve(listener, app).await.unwrap();
}

async fn list_books(State(state): State<AppState>) -> Json<Vec<Book>> {
    let books = state.lock().unwrap().clone();
    Json(books)
}

async fn get_book(
    Path(id): Path<u32>,
    State(state): State<AppState>,
) -> Result<Json<Book>, StatusCode> {
    let books = state.lock().unwrap();
    books.iter()
        .find(|b| b.id == id)
        .cloned()
        .map(Json)
        .ok_or(StatusCode::NOT_FOUND)
}

async fn create_book(
    State(state): State<AppState>,
    Json(create): Json<CreateBook>,
) -> (StatusCode, Json<Book>) {
    let mut books = state.lock().unwrap();
    let id = books.iter().map(|b| b.id).max().unwrap_or(0) + 1;

    let book = Book {
        id,
        title: create.title,
        author: create.author,
        isbn: create.isbn,
        available: true,
    };

    books.push(book.clone());
    (StatusCode::CREATED, Json(book))
}

async fn update_book(
    Path(id): Path<u32>,
    State(state): State<AppState>,
    Json(update): Json<UpdateBook>,
) -> Result<Json<Book>, StatusCode> {
    let mut books = state.lock().unwrap();

    books.iter_mut()
        .find(|b| b.id == id)
        .map(|book| {
            if let Some(title) = update.title {
                book.title = title;
            }
            if let Some(author) = update.author {
                book.author = author;
            }
            if let Some(available) = update.available {
                book.available = available;
            }
            Json(book.clone())
        })
        .ok_or(StatusCode::NOT_FOUND)
}

async fn delete_book(
    Path(id): Path<u32>,
    State(state): State<AppState>,
) -> StatusCode {
    let mut books = state.lock().unwrap();
    let len_before = books.len();
    books.retain(|b| b.id != id);

    if books.len() < len_before {
        StatusCode::NO_CONTENT
    } else {
        StatusCode::NOT_FOUND
    }
}
```
</details>

### Exercise 2: Input Validation

Add validation to the book creation:
- Title must not be empty
- ISBN must be exactly 13 characters
- Author must not be empty

Return 400 Bad Request with error message if validation fails.

<details>
<summary>Hint</summary>

Create a validation function that returns `Result<(), String>`:

```rust
fn validate_create_book(book: &CreateBook) -> Result<(), String> {
    if book.title.trim().is_empty() {
        return Err("Title cannot be empty".to_string());
    }
    if book.isbn.len() != 13 {
        return Err("ISBN must be 13 characters".to_string());
    }
    if book.author.trim().is_empty() {
        return Err("Author cannot be empty".to_string());
    }
    Ok(())
}
```

Use it in the handler before creating the book.
</details>

### Exercise 3: Custom Error Type

Create a custom error type for the Book API that handles:
- NotFound
- ValidationError(String)
- InternalError

Implement `IntoResponse` to return proper JSON error responses.

<details>
<summary>Solution</summary>

```rust
use serde::Serialize;

#[derive(Serialize)]
struct ErrorResponse {
    error: String,
    message: String,
}

enum BookApiError {
    NotFound,
    ValidationError(String),
    InternalError,
}

impl IntoResponse for BookApiError {
    fn into_response(self) -> Response {
        let (status, error, message) = match self {
            BookApiError::NotFound => (
                StatusCode::NOT_FOUND,
                "NOT_FOUND",
                "Book not found".to_string(),
            ),
            BookApiError::ValidationError(msg) => (
                StatusCode::BAD_REQUEST,
                "VALIDATION_ERROR",
                msg,
            ),
            BookApiError::InternalError => (
                StatusCode::INTERNAL_SERVER_ERROR,
                "INTERNAL_ERROR",
                "Something went wrong".to_string(),
            ),
        };

        (
            status,
            Json(ErrorResponse {
                error: error.to_string(),
                message,
            }),
        ).into_response()
    }
}

async fn create_book(
    State(state): State<AppState>,
    Json(create): Json<CreateBook>,
) -> Result<(StatusCode, Json<Book>), BookApiError> {
    // Validate
    if create.title.trim().is_empty() {
        return Err(BookApiError::ValidationError("Title is required".into()));
    }

    // Create book
    let mut books = state.lock().unwrap();
    let id = books.iter().map(|b| b.id).max().unwrap_or(0) + 1;

    let book = Book {
        id,
        title: create.title,
        author: create.author,
        isbn: create.isbn,
        available: true,
    };

    books.push(book.clone());
    Ok((StatusCode::CREATED, Json(book)))
}
```
</details>

## Key Takeaways

- ✅ `Json<T>` automatically serializes/deserializes JSON
- ✅ Use `Arc<Mutex<T>>` for shared state across handlers
- ✅ Return appropriate status codes (200, 201, 204, 400, 404, 500)
- ✅ Create custom error types implementing `IntoResponse`
- ✅ Validate input data before processing
- ✅ Keep Mutex locks held for minimal time
- ✅ Use `with_state()` to share data across handlers

**Next**: Building a complete RESTful API practice project!

---

**Progress**: Module 12, Lesson 4 complete (61/90+ lessons total)
