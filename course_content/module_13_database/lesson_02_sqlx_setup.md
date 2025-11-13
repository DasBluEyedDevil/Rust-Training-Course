# Module 13: Database Integration

# Lesson 2: Setting Up SQLx

## What is SQLx?

**SQLx** is a Rust SQL toolkit that provides:
- Async database access (works with tokio)
- Compile-time checked queries
- Connection pooling
- Support for PostgreSQL, MySQL, SQLite
- Type-safe query results

**Latest version: 0.8.6** (as of November 2025)

## Project Setup

### Create New Project

```bash
cargo new blog_db
cd blog_db
```

### Add Dependencies

Edit `Cargo.toml`:

```toml
[package]
name = "blog_db"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "chrono", "uuid"] }
chrono = { version = "0.4", features = ["serde"] }
uuid = { version = "1.0", features = ["v4", "serde"] }
dotenvy = "0.15"
```

**Dependency features explained:**
- `runtime-tokio` - Use tokio async runtime
- `postgres` - PostgreSQL support
- `chrono` - Date/time types
- `uuid` - UUID support for primary keys
- `dotenvy` - Load environment variables from .env file

### Install sqlx-cli

The sqlx command-line tool manages migrations:

```bash
cargo install sqlx-cli --no-default-features --features postgres
```

Verify installation:

```bash
sqlx --version
```

## Database Setup

### Create Database

```bash
# Using psql
createdb blog_dev

# Or using sqlx-cli
sqlx database create --database-url postgresql://postgres:password@localhost/blog_dev
```

### Create .env File

Create `.env` in your project root:

```env
DATABASE_URL=postgresql://postgres:your_password@localhost:5432/blog_dev
```

**Important:** Add `.env` to `.gitignore` to avoid committing passwords!

```.gitignore
/target
.env
```

## Your First Connection

Create `src/main.rs`:

```rust
use sqlx::postgres::PgPoolOptions;
use std::env;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    // Load .env file
    dotenvy::dotenv().ok();

    // Get database URL from environment
    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set in .env file");

    // Create connection pool
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await?;

    println!("✅ Connected to database!");

    // Test query
    let result: (i64,) = sqlx::query_as("SELECT $1 as value")
        .bind(42_i64)
        .fetch_one(&pool)
        .await?;

    println!("Test query result: {}", result.0);

    Ok(())
}
```

**Run it:**

```bash
cargo run
# Output:
# ✅ Connected to database!
# Test query result: 42
```

## Understanding Connection Pools

```rust
let pool = PgPoolOptions::new()
    .max_connections(5)
    .connect(&database_url)
    .await?;
```

**What is a connection pool?**
- Maintains a pool of reusable database connections
- Connections are expensive to create
- Pool reuses connections instead of creating new ones

**Configuration options:**
```rust
let pool = PgPoolOptions::new()
    .max_connections(10)           // Max concurrent connections
    .min_connections(2)            // Keep at least 2 connections open
    .acquire_timeout(Duration::from_secs(30))  // Wait max 30s for connection
    .connect(&database_url)
    .await?;
```

## Database Migrations

Migrations are version-controlled schema changes.

### Initialize Migrations

```bash
sqlx migrate add create_users_table
```

This creates: `migrations/TIMESTAMP_create_users_table.sql`

### Write Migration

Edit the generated file:

```sql
-- migrations/20251113_create_users_table.sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Add index for email lookups
CREATE INDEX idx_users_email ON users(email);
```

### Run Migration

```bash
sqlx migrate run
```

You should see:
```
Applied 20251113/create_users_table (...)
```

### Check Migration Status

```bash
sqlx migrate info
```

### Revert Migration (if needed)

```bash
sqlx migrate revert
```

## Basic Queries

### Insert Data

```rust
use sqlx::PgPool;

async fn create_user(
    pool: &PgPool,
    username: &str,
    email: &str,
    password_hash: &str,
) -> Result<i32, sqlx::Error> {
    let result = sqlx::query!(
        "INSERT INTO users (username, email, password_hash) VALUES ($1, $2, $3) RETURNING id",
        username,
        email,
        password_hash
    )
    .fetch_one(pool)
    .await?;

    Ok(result.id)
}
```

**The `query!` macro:**
- Validates SQL at compile time!
- Checks types match database schema
- Auto-generates type-safe result structs

### Select Data

```rust
#[derive(Debug)]
struct User {
    id: i32,
    username: String,
    email: String,
    password_hash: String,
    created_at: chrono::NaiveDateTime,
}

async fn get_user_by_email(
    pool: &PgPool,
    email: &str,
) -> Result<Option<User>, sqlx::Error> {
    let user = sqlx::query_as!(
        User,
        "SELECT id, username, email, password_hash, created_at FROM users WHERE email = $1",
        email
    )
    .fetch_optional(pool)
    .await?;

    Ok(user)
}
```

### Update Data

```rust
async fn update_username(
    pool: &PgPool,
    user_id: i32,
    new_username: &str,
) -> Result<(), sqlx::Error> {
    sqlx::query!(
        "UPDATE users SET username = $1 WHERE id = $2",
        new_username,
        user_id
    )
    .execute(pool)
    .await?;

    Ok(())
}
```

### Delete Data

```rust
async fn delete_user(
    pool: &PgPool,
    user_id: i32,
) -> Result<u64, sqlx::Error> {
    let result = sqlx::query!(
        "DELETE FROM users WHERE id = $1",
        user_id
    )
    .execute(pool)
    .await?;

    Ok(result.rows_affected())
}
```

## Complete Example

```rust
use sqlx::postgres::PgPoolOptions;
use sqlx::PgPool;
use std::env;

#[derive(Debug)]
struct User {
    id: i32,
    username: String,
    email: String,
    created_at: chrono::NaiveDateTime,
}

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    dotenvy::dotenv().ok();

    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");

    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await?;

    println!("✅ Connected to database!");

    // Create a user
    let user_id = create_user(&pool, "alice", "alice@example.com", "hashed_password").await?;
    println!("Created user with ID: {}", user_id);

    // Find user by email
    if let Some(user) = get_user_by_email(&pool, "alice@example.com").await? {
        println!("Found user: {:?}", user);
    }

    // Update username
    update_username(&pool, user_id, "alice_updated").await?;
    println!("Updated username");

    // List all users
    let users = list_users(&pool).await?;
    println!("All users: {:?}", users);

    Ok(())
}

async fn create_user(
    pool: &PgPool,
    username: &str,
    email: &str,
    password_hash: &str,
) -> Result<i32, sqlx::Error> {
    let result = sqlx::query!(
        "INSERT INTO users (username, email, password_hash) VALUES ($1, $2, $3) RETURNING id",
        username,
        email,
        password_hash
    )
    .fetch_one(pool)
    .await?;

    Ok(result.id)
}

async fn get_user_by_email(
    pool: &PgPool,
    email: &str,
) -> Result<Option<User>, sqlx::Error> {
    let user = sqlx::query_as!(
        User,
        r#"
        SELECT id, username, email, created_at
        FROM users
        WHERE email = $1
        "#,
        email
    )
    .fetch_optional(pool)
    .await?;

    Ok(user)
}

async fn update_username(
    pool: &PgPool,
    user_id: i32,
    new_username: &str,
) -> Result<(), sqlx::Error> {
    sqlx::query!(
        "UPDATE users SET username = $1 WHERE id = $2",
        new_username,
        user_id
    )
    .execute(pool)
    .await?;

    Ok(())
}

async fn list_users(pool: &PgPool) -> Result<Vec<User>, sqlx::Error> {
    let users = sqlx::query_as!(
        User,
        "SELECT id, username, email, created_at FROM users ORDER BY created_at DESC"
    )
    .fetch_all(pool)
    .await?;

    Ok(users)
}
```

## Query Types

### fetch_one - Expect exactly one row

```rust
let user = sqlx::query_as!(User, "SELECT * FROM users WHERE id = $1", id)
    .fetch_one(&pool)  // Returns Error if not found or multiple rows
    .await?;
```

### fetch_optional - Zero or one row

```rust
let maybe_user = sqlx::query_as!(User, "SELECT * FROM users WHERE email = $1", email)
    .fetch_optional(&pool)  // Returns Option<User>
    .await?;
```

### fetch_all - All matching rows

```rust
let users = sqlx::query_as!(User, "SELECT * FROM users")
    .fetch_all(&pool)  // Returns Vec<User>
    .await?;
```

### execute - Don't return rows

```rust
let result = sqlx::query!("DELETE FROM users WHERE id = $1", id)
    .execute(&pool)  // Returns PgQueryResult with rows_affected()
    .await?;

println!("Deleted {} rows", result.rows_affected());
```

## Handling NULL Values

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    published_at TIMESTAMP  -- Can be NULL
);
```

```rust
#[derive(Debug)]
struct Post {
    id: i32,
    title: String,
    content: String,
    published_at: Option<chrono::NaiveDateTime>,  // NULL becomes None
}
```

## Practice Exercises

### Exercise 1: Posts Table

Create a migration for a posts table with:
- id (serial primary key)
- user_id (foreign key to users)
- title (varchar 255, not null)
- content (text, not null)
- published (boolean, default false)
- created_at (timestamp, default now)

<details>
<summary>Solution</summary>

```bash
sqlx migrate add create_posts_table
```

```sql
-- migrations/TIMESTAMP_create_posts_table.sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    published BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_published ON posts(published);
```

```bash
sqlx migrate run
```
</details>

### Exercise 2: CRUD Functions for Posts

Implement these functions:

```rust
async fn create_post(pool: &PgPool, user_id: i32, title: &str, content: &str) -> Result<i32, sqlx::Error>;
async fn get_post(pool: &PgPool, post_id: i32) -> Result<Option<Post>, sqlx::Error>;
async fn list_user_posts(pool: &PgPool, user_id: i32) -> Result<Vec<Post>, sqlx::Error>;
async fn publish_post(pool: &PgPool, post_id: i32) -> Result<(), sqlx::Error>;
async fn delete_post(pool: &PgPool, post_id: i32) -> Result<u64, sqlx::Error>;
```

<details>
<summary>Solution</summary>

```rust
#[derive(Debug)]
struct Post {
    id: i32,
    user_id: i32,
    title: String,
    content: String,
    published: bool,
    created_at: chrono::NaiveDateTime,
}

async fn create_post(
    pool: &PgPool,
    user_id: i32,
    title: &str,
    content: &str,
) -> Result<i32, sqlx::Error> {
    let result = sqlx::query!(
        "INSERT INTO posts (user_id, title, content) VALUES ($1, $2, $3) RETURNING id",
        user_id,
        title,
        content
    )
    .fetch_one(pool)
    .await?;

    Ok(result.id)
}

async fn get_post(
    pool: &PgPool,
    post_id: i32,
) -> Result<Option<Post>, sqlx::Error> {
    let post = sqlx::query_as!(
        Post,
        "SELECT id, user_id, title, content, published, created_at FROM posts WHERE id = $1",
        post_id
    )
    .fetch_optional(pool)
    .await?;

    Ok(post)
}

async fn list_user_posts(
    pool: &PgPool,
    user_id: i32,
) -> Result<Vec<Post>, sqlx::Error> {
    let posts = sqlx::query_as!(
        Post,
        "SELECT id, user_id, title, content, published, created_at FROM posts WHERE user_id = $1 ORDER BY created_at DESC",
        user_id
    )
    .fetch_all(pool)
    .await?;

    Ok(posts)
}

async fn publish_post(
    pool: &PgPool,
    post_id: i32,
) -> Result<(), sqlx::Error> {
    sqlx::query!(
        "UPDATE posts SET published = TRUE WHERE id = $1",
        post_id
    )
    .execute(pool)
    .await?;

    Ok(())
}

async fn delete_post(
    pool: &PgPool,
    post_id: i32,
) -> Result<u64, sqlx::Error> {
    let result = sqlx::query!(
        "DELETE FROM posts WHERE id = $1",
        post_id
    )
    .execute(pool)
    .await?;

    Ok(result.rows_affected())
}
```
</details>

## Common Issues & Solutions

### Issue: "no DATABASE_URL environment variable"

**Solution:** Create `.env` file with your database URL

### Issue: "error returned from database: relation \"users\" does not exist"

**Solution:** Run migrations: `sqlx migrate run`

### Issue: Compile-time verification fails

**Solution:** Make sure database is running and `DATABASE_URL` is set during build

Or use offline mode:
```bash
cargo sqlx prepare  # Generate sqlx-data.json
cargo build --offline
```

### Issue: "password authentication failed"

**Solution:** Check your DATABASE_URL username/password

## Key Takeaways

- ✅ SQLx provides async, type-safe database access
- ✅ Connection pools manage database connections efficiently
- ✅ Migrations version-control schema changes
- ✅ `query!` macro validates SQL at compile time
- ✅ Use `fetch_one`, `fetch_optional`, `fetch_all` based on expected rows
- ✅ SQLx supports PostgreSQL, MySQL, and SQLite
- ✅ Always use `.env` for sensitive configuration

**Next**: Integrating SQLx with Axum for database-backed APIs!

---

**Progress**: Module 13, Lesson 2 complete (64/90+ lessons total)
