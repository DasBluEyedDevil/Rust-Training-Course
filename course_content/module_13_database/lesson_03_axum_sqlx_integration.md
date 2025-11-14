# Module 13: Database Integration

# Lesson 3: Axum + SQLx Integration

## Combining Web API + Database

Now let's build a real API backed by PostgreSQL instead of in-memory storage!

## Project Setup

```bash
cargo new blog_api_db
cd blog_api_db
```

### Dependencies

```toml
[package]
name = "blog_api_db"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.8.x"
tokio = { version = "1.x", features = ["full"] }
sqlx = { version = "0.8.x", features = ["runtime-tokio", "postgres", "chrono"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
chrono = { version = "0.4", features = ["serde"] }
dotenvy = "0.15"
```

### Database Setup

Create `.env`:

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/blog_dev
```

## Database Schema

Create migrations:

```bash
sqlx migrate add create_schema
```

Edit `migrations/TIMESTAMP_create_schema.sql`:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    published BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_published ON posts(published);

-- Insert sample data
INSERT INTO users (username, email) VALUES
    ('alice', 'alice@example.com'),
    ('bob', 'bob@example.com');

INSERT INTO posts (user_id, title, content, published) VALUES
    (1, 'Getting Started with Rust', 'Rust is amazing...', TRUE),
    (1, 'Building Web APIs', 'Let me show you how...', TRUE),
    (2, 'My First Post', 'Hello world!', FALSE);
```

Run migration:

```bash
sqlx migrate run
```

## Sharing Database Pool in Axum

The key is passing the `PgPool` to handlers using Axum's `State`:

```rust
use axum::{
    Router,
    extract::State,
};
use sqlx::PgPool;

#[tokio::main]
async fn main() {
    let pool = /* create pool */;

    let app = Router::new()
        .route("/users", get(list_users))
        .with_state(pool);  // Share pool with all handlers

    // ...
}

async fn list_users(
    State(pool): State<PgPool>,  // Extract pool in handler
) -> Json<Vec<User>> {
    // Use pool to query database
}
```

## Complete Example

### Project Structure

```
src/
├── main.rs
├── models.rs
├── handlers/
│   ├── mod.rs
│   ├── users.rs
│   └── posts.rs
```

### src/models.rs

```rust
use serde::{Deserialize, Serialize};
use chrono::NaiveDateTime;

#[derive(Debug, Serialize, Deserialize)]
pub struct User {
    pub id: i32,
    pub username: String,
    pub email: String,
    pub created_at: NaiveDateTime,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct Post {
    pub id: i32,
    pub user_id: i32,
    pub title: String,
    pub content: String,
    pub published: bool,
    pub created_at: NaiveDateTime,
    pub updated_at: NaiveDateTime,
}

#[derive(Debug, Deserialize)]
pub struct CreatePost {
    pub user_id: i32,
    pub title: String,
    pub content: String,
}

#[derive(Debug, Deserialize)]
pub struct UpdatePost {
    pub title: Option<String>,
    pub content: Option<String>,
    pub published: Option<bool>,
}
```

### src/handlers/users.rs

```rust
use axum::{
    extract::State,
    Json,
};
use sqlx::PgPool;

use crate::models::User;

pub async fn list_users(
    State(pool): State<PgPool>,
) -> Result<Json<Vec<User>>, (axum::http::StatusCode, String)> {
    let users = sqlx::query_as!(
        User,
        "SELECT id, username, email, created_at FROM users ORDER BY created_at DESC"
    )
    .fetch_all(&pool)
    .await
    .map_err(internal_error)?;

    Ok(Json(users))
}

pub async fn get_user(
    State(pool): State<PgPool>,
    axum::extract::Path(id): axum::extract::Path<i32>,
) -> Result<Json<User>, (axum::http::StatusCode, String)> {
    let user = sqlx::query_as!(
        User,
        "SELECT id, username, email, created_at FROM users WHERE id = $1",
        id
    )
    .fetch_optional(&pool)
    .await
    .map_err(internal_error)?
    .ok_or_else(|| {
        (
            axum::http::StatusCode::NOT_FOUND,
            "User not found".to_string(),
        )
    })?;

    Ok(Json(user))
}

fn internal_error<E>(err: E) -> (axum::http::StatusCode, String)
where
    E: std::error::Error,
{
    (axum::http::StatusCode::INTERNAL_SERVER_ERROR, err.to_string())
}
```

### src/handlers/posts.rs

```rust
use axum::{
    extract::{Path, Query, State},
    http::StatusCode,
    Json,
};
use serde::Deserialize;
use sqlx::PgPool;

use crate::models::{Post, CreatePost, UpdatePost};

#[derive(Deserialize)]
pub struct ListQuery {
    #[serde(default)]
    pub published: Option<bool>,
}

pub async fn list_posts(
    State(pool): State<PgPool>,
    Query(query): Query<ListQuery>,
) -> Result<Json<Vec<Post>>, (StatusCode, String)> {
    let posts = if let Some(published) = query.published {
        sqlx::query_as!(
            Post,
            "SELECT * FROM posts WHERE published = $1 ORDER BY created_at DESC",
            published
        )
        .fetch_all(&pool)
        .await
    } else {
        sqlx::query_as!(
            Post,
            "SELECT * FROM posts ORDER BY created_at DESC"
        )
        .fetch_all(&pool)
        .await
    }
    .map_err(internal_error)?;

    Ok(Json(posts))
}

pub async fn get_post(
    State(pool): State<PgPool>,
    Path(id): Path<i32>,
) -> Result<Json<Post>, (StatusCode, String)> {
    let post = sqlx::query_as!(
        Post,
        "SELECT * FROM posts WHERE id = $1",
        id
    )
    .fetch_optional(&pool)
    .await
    .map_err(internal_error)?
    .ok_or_else(|| (StatusCode::NOT_FOUND, "Post not found".to_string()))?;

    Ok(Json(post))
}

pub async fn create_post(
    State(pool): State<PgPool>,
    Json(create): Json<CreatePost>,
) -> Result<(StatusCode, Json<Post>), (StatusCode, String)> {
    // Verify user exists
    let user_exists = sqlx::query!("SELECT id FROM users WHERE id = $1", create.user_id)
        .fetch_optional(&pool)
        .await
        .map_err(internal_error)?
        .is_some();

    if !user_exists {
        return Err((StatusCode::BAD_REQUEST, "User does not exist".to_string()));
    }

    let post = sqlx::query_as!(
        Post,
        r#"
        INSERT INTO posts (user_id, title, content)
        VALUES ($1, $2, $3)
        RETURNING id, user_id, title, content, published, created_at, updated_at
        "#,
        create.user_id,
        create.title,
        create.content
    )
    .fetch_one(&pool)
    .await
    .map_err(internal_error)?;

    Ok((StatusCode::CREATED, Json(post)))
}

pub async fn update_post(
    State(pool): State<PgPool>,
    Path(id): Path<i32>,
    Json(update): Json<UpdatePost>,
) -> Result<Json<Post>, (StatusCode, String)> {
    // Get existing post
    let mut post = sqlx::query_as!(
        Post,
        "SELECT * FROM posts WHERE id = $1",
        id
    )
    .fetch_optional(&pool)
    .await
    .map_err(internal_error)?
    .ok_or_else(|| (StatusCode::NOT_FOUND, "Post not found".to_string()))?;

    // Apply updates
    if let Some(title) = update.title {
        post.title = title;
    }
    if let Some(content) = update.content {
        post.content = content;
    }
    if let Some(published) = update.published {
        post.published = published;
    }

    // Save to database
    let updated_post = sqlx::query_as!(
        Post,
        r#"
        UPDATE posts
        SET title = $1, content = $2, published = $3, updated_at = NOW()
        WHERE id = $4
        RETURNING id, user_id, title, content, published, created_at, updated_at
        "#,
        post.title,
        post.content,
        post.published,
        id
    )
    .fetch_one(&pool)
    .await
    .map_err(internal_error)?;

    Ok(Json(updated_post))
}

pub async fn delete_post(
    State(pool): State<PgPool>,
    Path(id): Path<i32>,
) -> Result<StatusCode, (StatusCode, String)> {
    let result = sqlx::query!("DELETE FROM posts WHERE id = $1", id)
        .execute(&pool)
        .await
        .map_err(internal_error)?;

    if result.rows_affected() == 0 {
        return Err((StatusCode::NOT_FOUND, "Post not found".to_string()));
    }

    Ok(StatusCode::NO_CONTENT)
}

pub async fn list_user_posts(
    State(pool): State<PgPool>,
    Path(user_id): Path<i32>,
) -> Result<Json<Vec<Post>>, (StatusCode, String)> {
    let posts = sqlx::query_as!(
        Post,
        "SELECT * FROM posts WHERE user_id = $1 ORDER BY created_at DESC",
        user_id
    )
    .fetch_all(&pool)
    .await
    .map_err(internal_error)?;

    Ok(Json(posts))
}

fn internal_error<E>(err: E) -> (StatusCode, String)
where
    E: std::error::Error,
{
    (StatusCode::INTERNAL_SERVER_ERROR, err.to_string())
}
```

### src/handlers/mod.rs

```rust
pub mod users;
pub mod posts;
```

### src/main.rs

```rust
mod models;
mod handlers;

use axum::{
    routing::{get, post, put, delete},
    Router,
};
use sqlx::postgres::PgPoolOptions;
use std::env;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Load environment variables
    dotenvy::dotenv().ok();

    // Get database URL
    let database_url = env::var("DATABASE_URL")?;

    // Create connection pool
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await?;

    println!("✅ Connected to database");

    // Build router
    let app = Router::new()
        .route("/", get(|| async { "Blog API with Database" }))
        // User routes
        .route("/users", get(handlers::users::list_users))
        .route("/users/{id}", get(handlers::users::get_user))
        // Post routes
        .route("/posts", get(handlers::posts::list_posts).post(handlers::posts::create_post))
        .route("/posts/{id}",
            get(handlers::posts::get_post)
            .put(handlers::posts::update_post)
            .delete(handlers::posts::delete_post)
        )
        .route("/users/{user_id}/posts", get(handlers::posts::list_user_posts))
        .with_state(pool);

    // Start server
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;

    println!("🚀 Server running on http://localhost:3000");
    println!("\n📚 Endpoints:");
    println!("  GET    /users");
    println!("  GET    /users/{{id}}");
    println!("  GET    /posts");
    println!("  GET    /posts?published=true");
    println!("  GET    /posts/{{id}}");
    println!("  POST   /posts");
    println!("  PUT    /posts/{{id}}");
    println!("  DELETE /posts/{{id}}");
    println!("  GET    /users/{{user_id}}/posts");

    axum::serve(listener, app).await?;

    Ok(())
}
```

## Testing the API

### List all users

```bash
curl http://localhost:3000/users
```

### Get specific user

```bash
curl http://localhost:3000/users/1
```

### List all posts

```bash
curl http://localhost:3000/posts
```

### List only published posts

```bash
curl "http://localhost:3000/posts?published=true"
```

### Get user's posts

```bash
curl http://localhost:3000/users/1/posts
```

### Create a post

```bash
curl -X POST http://localhost:3000/posts \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": 1,
    "title": "New Database Post",
    "content": "This is stored in PostgreSQL!"
  }'
```

### Update a post

```bash
curl -X PUT http://localhost:3000/posts/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Title",
    "published": true
  }'
```

### Delete a post

```bash
curl -X DELETE http://localhost:3000/posts/3
```

## Error Handling Best Practices

### Custom Error Type

Create `src/error.rs`:

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
}

pub enum ApiError {
    NotFound(String),
    BadRequest(String),
    InternalError,
    DatabaseError(String),
}

impl IntoResponse for ApiError {
    fn into_response(self) -> Response {
        let (status, message) = match self {
            ApiError::NotFound(msg) => (StatusCode::NOT_FOUND, msg),
            ApiError::BadRequest(msg) => (StatusCode::BAD_REQUEST, msg),
            ApiError::DatabaseError(msg) => (StatusCode::INTERNAL_SERVER_ERROR, format!("Database error: {}", msg)),
            ApiError::InternalError => (StatusCode::INTERNAL_SERVER_ERROR, "Internal error".to_string()),
        };

        (status, Json(ErrorResponse { error: message })).into_response()
    }
}

impl From<sqlx::Error> for ApiError {
    fn from(err: sqlx::Error) -> Self {
        match err {
            sqlx::Error::RowNotFound => ApiError::NotFound("Resource not found".to_string()),
            _ => ApiError::DatabaseError(err.to_string()),
        }
    }
}
```

Update handlers to use `ApiError`:

```rust
use crate::error::ApiError;

pub async fn get_post(
    State(pool): State<PgPool>,
    Path(id): Path<i32>,
) -> Result<Json<Post>, ApiError> {
    let post = sqlx::query_as!(Post, "SELECT * FROM posts WHERE id = $1", id)
        .fetch_one(&pool)
        .await?;  // sqlx::Error automatically converts to ApiError

    Ok(Json(post))
}
```

## Transactions

For operations that need to succeed or fail together:

```rust
use sqlx::Acquire;

pub async fn transfer_post_ownership(
    State(pool): State<PgPool>,
    Path((post_id, new_user_id)): Path<(i32, i32)>,
) -> Result<StatusCode, ApiError> {
    let mut tx = pool.begin().await?;

    // Check new user exists
    sqlx::query!("SELECT id FROM users WHERE id = $1", new_user_id)
        .fetch_one(&mut *tx)
        .await?;

    // Update post
    sqlx::query!("UPDATE posts SET user_id = $1 WHERE id = $2", new_user_id, post_id)
        .execute(&mut *tx)
        .await?;

    // Commit transaction
    tx.commit().await?;

    Ok(StatusCode::OK)
}
```

## Practice Exercises

### Exercise 1: Add Comments

Create a comments table and API:
- Migration for comments table
- Models for Comment
- Handlers for creating and listing comments on posts

<details>
<summary>Hint</summary>

Migration:
```sql
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

Routes:
- `GET /posts/{post_id}/comments`
- `POST /posts/{post_id}/comments`
</details>

### Exercise 2: Add Pagination

Implement pagination for the posts list:
- Accept `page` and `limit` query parameters
- Use SQL `LIMIT` and `OFFSET`
- Return total count in response

<details>
<summary>Solution Hint</summary>

```rust
#[derive(Deserialize)]
struct Pagination {
    #[serde(default = "default_page")]
    page: i64,
    #[serde(default = "default_limit")]
    limit: i64,
}

fn default_page() -> i64 { 0 }
fn default_limit() -> i64 { 10 }

let offset = query.page * query.limit;

let posts = sqlx::query_as!(
    Post,
    "SELECT * FROM posts ORDER BY created_at DESC LIMIT $1 OFFSET $2",
    query.limit,
    offset
)
.fetch_all(&pool)
.await?;
```
</details>

### Exercise 3: Search Posts

Add `/posts/search?q=rust` endpoint that searches titles and content.

<details>
<summary>Solution</summary>

```rust
pub async fn search_posts(
    State(pool): State<PgPool>,
    Query(query): Query<SearchQuery>,
) -> Result<Json<Vec<Post>>, ApiError> {
    let search_term = format!("%{}%", query.q);

    let posts = sqlx::query_as!(
        Post,
        r#"
        SELECT * FROM posts
        WHERE title ILIKE $1 OR content ILIKE $1
        ORDER BY created_at DESC
        "#,
        search_term
    )
    .fetch_all(&pool)
    .await?;

    Ok(Json(posts))
}
```
</details>

## Key Takeaways

- ✅ Share `PgPool` across handlers with Axum's `State`
- ✅ Use `sqlx::query_as!` for type-safe database queries
- ✅ Convert `sqlx::Error` to custom API errors
- ✅ Use transactions for multi-step operations
- ✅ Validate foreign keys before inserting
- ✅ Return appropriate status codes (200, 201, 404, etc.)
- ✅ Database persistence survives server restarts!

**Next**: Advanced database patterns and optimization!

---

**Progress**: Module 13, Lesson 3 complete (65/90+ lessons total)
