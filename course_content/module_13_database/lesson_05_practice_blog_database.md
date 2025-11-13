# Module 13: Database Integration

# Lesson 5: Practice Project — Full-Stack Blog with Database

## Project Overview

Build a complete blog platform with:
- Users, Posts, Comments, Tags
- Full CRUD operations
- Relationships and joins
- Search functionality
- Statistics dashboard
- All backed by PostgreSQL!

This integrates everything from Modules 12 and 13.

## Project Setup

```bash
cargo new fullstack_blog
cd fullstack_blog
```

### Dependencies

```toml
[package]
name = "fullstack_blog"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.8"
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "chrono"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
chrono = { version = "0.4", features = ["serde"] }
dotenvy = "0.15"
```

### Environment Setup

Create `.env`:

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/fullstack_blog
```

## Database Schema

### Create Database

```bash
sqlx database create
```

### Migration 1: Core Tables

```bash
sqlx migrate add create_core_tables
```

Edit `migrations/TIMESTAMP_create_core_tables.sql`:

```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    bio TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Posts table
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    content TEXT NOT NULL,
    excerpt VARCHAR(500),
    published BOOLEAN DEFAULT FALSE,
    view_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Comments table
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Tags table
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL
);

-- Post-Tag junction table
CREATE TABLE post_tags (
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    tag_id INTEGER REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);

-- Indexes
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_published ON posts(published);
CREATE INDEX idx_posts_slug ON posts(slug);
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);
CREATE INDEX idx_post_tags_post_id ON post_tags(post_id);
CREATE INDEX idx_post_tags_tag_id ON post_tags(tag_id);

-- Full-text search
ALTER TABLE posts ADD COLUMN search_vector tsvector;
CREATE INDEX idx_posts_search ON posts USING GIN(search_vector);

CREATE FUNCTION posts_search_trigger() RETURNS trigger AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('english', COALESCE(NEW.title, '')), 'A') ||
        setweight(to_tsvector('english', COALESCE(NEW.content, '')), 'B');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER posts_search_update
    BEFORE INSERT OR UPDATE ON posts
    FOR EACH ROW EXECUTE FUNCTION posts_search_trigger();
```

### Migration 2: Sample Data

```bash
sqlx migrate add seed_data
```

Edit `migrations/TIMESTAMP_seed_data.sql`:

```sql
-- Insert users
INSERT INTO users (username, email, bio) VALUES
    ('alice', 'alice@example.com', 'Rust enthusiast and blogger'),
    ('bob', 'bob@example.com', 'Full-stack developer'),
    ('charlie', 'charlie@example.com', 'Systems programmer');

-- Insert tags
INSERT INTO tags (name, slug) VALUES
    ('Rust', 'rust'),
    ('Web Development', 'web-development'),
    ('Tutorial', 'tutorial'),
    ('Backend', 'backend'),
    ('Axum', 'axum');

-- Insert posts
INSERT INTO posts (user_id, title, slug, content, excerpt, published) VALUES
    (1, 'Getting Started with Rust', 'getting-started-rust',
     'Rust is a systems programming language focused on safety, speed, and concurrency...',
     'An introduction to Rust programming',
     true),
    (1, 'Building Web APIs with Axum', 'building-web-apis-axum',
     'Axum is a modern web framework built on Tokio, Tower, and Hyper...',
     'Learn how to build REST APIs with Axum',
     true),
    (2, 'Database Integration with SQLx', 'database-integration-sqlx',
     'SQLx provides async, type-safe database access for Rust...',
     'Complete guide to using SQLx',
     true),
    (3, 'Draft Post', 'draft-post',
     'This is a draft post...',
     'A work in progress',
     false);

-- Link posts to tags
INSERT INTO post_tags (post_id, tag_id) VALUES
    (1, 1), (1, 3),  -- Getting Started with Rust: Rust, Tutorial
    (2, 1), (2, 2), (2, 4), (2, 5),  -- Building Web APIs: Rust, Web Dev, Backend, Axum
    (3, 1), (3, 2), (3, 4);  -- Database Integration: Rust, Web Dev, Backend

-- Insert comments
INSERT INTO comments (post_id, user_id, content) VALUES
    (1, 2, 'Great introduction! Very helpful.'),
    (1, 3, 'Looking forward to more Rust content!'),
    (2, 3, 'Axum is amazing. Thanks for this guide!');
```

Run migrations:

```bash
sqlx migrate run
```

## Project Structure

```
src/
├── main.rs
├── models.rs
├── errors.rs
├── handlers/
│   ├── mod.rs
│   ├── users.rs
│   ├── posts.rs
│   ├── comments.rs
│   ├── tags.rs
│   └── stats.rs
```

## Implementation

### src/models.rs

```rust
use serde::{Deserialize, Serialize};
use chrono::NaiveDateTime;

#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
pub struct User {
    pub id: i32,
    pub username: String,
    pub email: String,
    pub bio: Option<String>,
    pub created_at: NaiveDateTime,
}

#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
pub struct Post {
    pub id: i32,
    pub user_id: i32,
    pub title: String,
    pub slug: String,
    pub content: String,
    pub excerpt: Option<String>,
    pub published: bool,
    pub view_count: i32,
    pub created_at: NaiveDateTime,
    pub updated_at: NaiveDateTime,
}

#[derive(Debug, Serialize)]
pub struct PostWithAuthor {
    #[serde(flatten)]
    pub post: Post,
    pub author: String,
    pub tags: Vec<String>,
    pub comment_count: i64,
}

#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
pub struct Comment {
    pub id: i32,
    pub post_id: i32,
    pub user_id: i32,
    pub content: String,
    pub created_at: NaiveDateTime,
}

#[derive(Debug, Serialize)]
pub struct CommentWithAuthor {
    #[serde(flatten)]
    pub comment: Comment,
    pub author: String,
}

#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
pub struct Tag {
    pub id: i32,
    pub name: String,
    pub slug: String,
}

#[derive(Debug, Deserialize)]
pub struct CreatePost {
    pub title: String,
    pub content: String,
    pub excerpt: Option<String>,
    pub published: bool,
    pub tags: Vec<String>,
}

#[derive(Debug, Deserialize)]
pub struct UpdatePost {
    pub title: Option<String>,
    pub content: Option<String>,
    pub excerpt: Option<String>,
    pub published: Option<bool>,
    pub tags: Option<Vec<String>>,
}

#[derive(Debug, Deserialize)]
pub struct CreateComment {
    pub content: String,
}

#[derive(Debug, Serialize)]
pub struct Stats {
    pub total_users: i64,
    pub total_posts: i64,
    pub published_posts: i64,
    pub draft_posts: i64,
    pub total_comments: i64,
    pub total_tags: i64,
}
```

### src/errors.rs

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
    InternalError(String),
}

impl IntoResponse for ApiError {
    fn into_response(self) -> Response {
        let (status, message) = match self {
            ApiError::NotFound(msg) => (StatusCode::NOT_FOUND, msg),
            ApiError::BadRequest(msg) => (StatusCode::BAD_REQUEST, msg),
            ApiError::InternalError(msg) => (StatusCode::INTERNAL_SERVER_ERROR, msg),
        };

        (status, Json(ErrorResponse { error: message })).into_response()
    }
}

impl From<sqlx::Error> for ApiError {
    fn from(err: sqlx::Error) -> Self {
        match err {
            sqlx::Error::RowNotFound => ApiError::NotFound("Resource not found".to_string()),
            _ => ApiError::InternalError(err.to_string()),
        }
    }
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

use crate::{
    models::{Post, PostWithAuthor, CreatePost, UpdatePost},
    errors::ApiError,
};

#[derive(Deserialize)]
pub struct ListQuery {
    pub tag: Option<String>,
    pub published: Option<bool>,
    #[serde(default)]
    pub page: i64,
    #[serde(default = "default_limit")]
    pub limit: i64,
}

fn default_limit() -> i64 { 10 }

pub async fn list_posts(
    State(pool): State<PgPool>,
    Query(query): Query<ListQuery>,
) -> Result<Json<Vec<PostWithAuthor>>, ApiError> {
    let offset = query.page * query.limit;

    let posts = if let Some(tag_slug) = query.tag {
        sqlx::query_as!(
            PostWithAuthor,
            r#"
            SELECT
                p.id, p.user_id, p.title, p.slug, p.content, p.excerpt,
                p.published, p.view_count, p.created_at, p.updated_at,
                u.username as author,
                ARRAY_AGG(t.name) FILTER (WHERE t.name IS NOT NULL) as "tags!: Vec<String>",
                COUNT(DISTINCT c.id) as "comment_count!"
            FROM posts p
            JOIN users u ON p.user_id = u.id
            LEFT JOIN post_tags pt ON p.id = pt.post_id
            LEFT JOIN tags t ON pt.tag_id = t.id
            LEFT JOIN comments c ON p.id = c.post_id
            WHERE p.published = COALESCE($1, p.published)
              AND EXISTS (
                  SELECT 1 FROM post_tags pt2
                  JOIN tags t2 ON pt2.tag_id = t2.id
                  WHERE pt2.post_id = p.id AND t2.slug = $2
              )
            GROUP BY p.id, p.user_id, p.title, p.slug, p.content, p.excerpt,
                     p.published, p.view_count, p.created_at, p.updated_at, u.username
            ORDER BY p.created_at DESC
            LIMIT $3 OFFSET $4
            "#,
            query.published,
            tag_slug,
            query.limit,
            offset
        )
        .fetch_all(&pool)
        .await?
    } else {
        sqlx::query_as!(
            PostWithAuthor,
            r#"
            SELECT
                p.id, p.user_id, p.title, p.slug, p.content, p.excerpt,
                p.published, p.view_count, p.created_at, p.updated_at,
                u.username as author,
                COALESCE(ARRAY_AGG(t.name) FILTER (WHERE t.name IS NOT NULL), '{}') as "tags!: Vec<String>",
                COUNT(DISTINCT c.id) as "comment_count!"
            FROM posts p
            JOIN users u ON p.user_id = u.id
            LEFT JOIN post_tags pt ON p.id = pt.post_id
            LEFT JOIN tags t ON pt.tag_id = t.id
            LEFT JOIN comments c ON p.id = c.post_id
            WHERE p.published = COALESCE($1, p.published)
            GROUP BY p.id, u.username
            ORDER BY p.created_at DESC
            LIMIT $2 OFFSET $3
            "#,
            query.published,
            query.limit,
            offset
        )
        .fetch_all(&pool)
        .await?
    };

    Ok(Json(posts))
}

pub async fn get_post(
    State(pool): State<PgPool>,
    Path(slug): Path<String>,
) -> Result<Json<PostWithAuthor>, ApiError> {
    // Increment view count
    sqlx::query!("UPDATE posts SET view_count = view_count + 1 WHERE slug = $1", slug)
        .execute(&pool)
        .await?;

    let post = sqlx::query_as!(
        PostWithAuthor,
        r#"
        SELECT
            p.id, p.user_id, p.title, p.slug, p.content, p.excerpt,
            p.published, p.view_count, p.created_at, p.updated_at,
            u.username as author,
            COALESCE(ARRAY_AGG(t.name) FILTER (WHERE t.name IS NOT NULL), '{}') as "tags!: Vec<String>",
            COUNT(DISTINCT c.id) as "comment_count!"
        FROM posts p
        JOIN users u ON p.user_id = u.id
        LEFT JOIN post_tags pt ON p.id = pt.post_id
        LEFT JOIN tags t ON pt.tag_id = t.id
        LEFT JOIN comments c ON p.id = c.post_id
        WHERE p.slug = $1
        GROUP BY p.id, u.username
        "#,
        slug
    )
    .fetch_one(&pool)
    .await?;

    Ok(Json(post))
}

pub async fn search_posts(
    State(pool): State<PgPool>,
    Query(query): Query<SearchQuery>,
) -> Result<Json<Vec<PostWithAuthor>>, ApiError> {
    let posts = sqlx::query_as!(
        PostWithAuthor,
        r#"
        SELECT
            p.id, p.user_id, p.title, p.slug, p.content, p.excerpt,
            p.published, p.view_count, p.created_at, p.updated_at,
            u.username as author,
            COALESCE(ARRAY_AGG(t.name) FILTER (WHERE t.name IS NOT NULL), '{}') as "tags!: Vec<String>",
            COUNT(DISTINCT c.id) as "comment_count!"
        FROM posts p
        JOIN users u ON p.user_id = u.id
        LEFT JOIN post_tags pt ON p.id = pt.post_id
        LEFT JOIN tags t ON pt.tag_id = t.id
        LEFT JOIN comments c ON p.id = c.post_id
        WHERE p.search_vector @@ plainto_tsquery('english', $1)
          AND p.published = true
        GROUP BY p.id, u.username
        ORDER BY ts_rank(p.search_vector, plainto_tsquery('english', $1)) DESC
        LIMIT 20
        "#,
        query.q
    )
    .fetch_all(&pool)
    .await?;

    Ok(Json(posts))
}

#[derive(Deserialize)]
pub struct SearchQuery {
    pub q: String,
}

fn slugify(s: &str) -> String {
    s.to_lowercase()
        .chars()
        .map(|c| if c.is_alphanumeric() { c } else { '-' })
        .collect::<String>()
        .split('-')
        .filter(|s| !s.is_empty())
        .collect::<Vec<&str>>()
        .join("-")
}

pub async fn create_post(
    State(pool): State<PgPool>,
    Path(user_id): Path<i32>,
    Json(create): Json<CreatePost>,
) -> Result<(StatusCode, Json<Post>), ApiError> {
    let slug = slugify(&create.title);

    let mut tx = pool.begin().await?;

    // Create post
    let post = sqlx::query_as!(
        Post,
        r#"
        INSERT INTO posts (user_id, title, slug, content, excerpt, published)
        VALUES ($1, $2, $3, $4, $5, $6)
        RETURNING *
        "#,
        user_id,
        create.title,
        slug,
        create.content,
        create.excerpt,
        create.published
    )
    .fetch_one(&mut *tx)
    .await?;

    // Add tags
    for tag_name in create.tags {
        let tag_slug = slugify(&tag_name);
        let tag = sqlx::query!(
            "INSERT INTO tags (name, slug) VALUES ($1, $2) ON CONFLICT (slug) DO UPDATE SET name = $1 RETURNING id",
            tag_name,
            tag_slug
        )
        .fetch_one(&mut *tx)
        .await?;

        sqlx::query!(
            "INSERT INTO post_tags (post_id, tag_id) VALUES ($1, $2)",
            post.id,
            tag.id
        )
        .execute(&mut *tx)
        .await?;
    }

    tx.commit().await?;

    Ok((StatusCode::CREATED, Json(post)))
}
```

### src/handlers/stats.rs

```rust
use axum::{extract::State, Json};
use sqlx::PgPool;

use crate::{models::Stats, errors::ApiError};

pub async fn get_stats(State(pool): State<PgPool>) -> Result<Json<Stats>, ApiError> {
    let stats = sqlx::query!(
        r#"
        SELECT
            (SELECT COUNT(*) FROM users) as "total_users!",
            (SELECT COUNT(*) FROM posts) as "total_posts!",
            (SELECT COUNT(*) FROM posts WHERE published = true) as "published_posts!",
            (SELECT COUNT(*) FROM posts WHERE published = false) as "draft_posts!",
            (SELECT COUNT(*) FROM comments) as "total_comments!",
            (SELECT COUNT(*) FROM tags) as "total_tags!"
        "#
    )
    .fetch_one(&pool)
    .await?;

    Ok(Json(Stats {
        total_users: stats.total_users,
        total_posts: stats.total_posts,
        published_posts: stats.published_posts,
        draft_posts: stats.draft_posts,
        total_comments: stats.total_comments,
        total_tags: stats.total_tags,
    }))
}
```

### src/main.rs

```rust
mod models;
mod errors;
mod handlers;

use axum::{
    routing::{get, post},
    Router,
};
use sqlx::postgres::PgPoolOptions;
use std::env;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    dotenvy::dotenv().ok();

    let database_url = env::var("DATABASE_URL")?;

    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await?;

    println!("✅ Connected to database");

    let app = Router::new()
        .route("/", get(|| async { "Full-Stack Blog API" }))
        .route("/posts", get(handlers::posts::list_posts))
        .route("/posts/search", get(handlers::posts::search_posts))
        .route("/posts/{slug}", get(handlers::posts::get_post))
        .route("/users/{user_id}/posts", post(handlers::posts::create_post))
        .route("/stats", get(handlers::stats::get_stats))
        .with_state(pool);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;

    println!("🚀 Server running on http://localhost:3000");
    println!("\n📚 Endpoints:");
    println!("  GET  /posts");
    println!("  GET  /posts?tag=rust&published=true");
    println!("  GET  /posts/{{slug}}");
    println!("  GET  /posts/search?q=rust");
    println!("  POST /users/{{user_id}}/posts");
    println!("  GET  /stats");

    axum::serve(listener, app).await?;

    Ok(())
}
```

## Testing

```bash
# List all published posts
curl "http://localhost:3000/posts?published=true"

# Filter by tag
curl "http://localhost:3000/posts?tag=rust"

# Get single post
curl http://localhost:3000/posts/getting-started-rust

# Search
curl "http://localhost:3000/posts/search?q=axum"

# Get stats
curl http://localhost:3000/stats

# Create post
curl -X POST http://localhost:3000/users/1/posts \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My New Post",
    "content": "This is the full content...",
    "excerpt": "A brief summary",
    "published": true,
    "tags": ["Rust", "Tutorial"]
  }'
```

## Challenge Extensions

Add these features yourself:

1. **User endpoints** - List users, get user profile
2. **Comment endpoints** - Add, list, delete comments
3. **Update/delete posts** - PUT and DELETE for posts
4. **Popular posts** - Endpoint for most viewed posts
5. **Trending tags** - Tags ordered by post count
6. **Related posts** - Posts with similar tags

## Key Takeaways

- ✅ Complete full-stack application with database
- ✅ Complex queries with joins and aggregations
- ✅ Many-to-many relationships (posts and tags)
- ✅ Full-text search
- ✅ View counting and statistics
- ✅ Proper error handling
- ✅ Transaction management
- ✅ Production-ready patterns

**Next Module**: Authentication & Security!

---

**Progress**: Module 13 complete! (67/90+ lessons total)
