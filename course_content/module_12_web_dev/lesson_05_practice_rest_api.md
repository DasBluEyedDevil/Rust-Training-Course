# Module 12: Web Development with Rust

# Lesson 5: Practice Project — Blog REST API

## Project Overview

Build a complete **Blog REST API** with:
- Posts (create, read, update, delete, list)
- Comments on posts
- Tags for categorization
- Filtering and pagination
- Proper error handling
- Input validation

This project integrates everything you've learned in Module 12!

## Project Setup

```bash
cargo new blog_api
cd blog_api
```

### Dependencies

```toml
[package]
name = "blog_api"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.8.x"
tokio = { version = "1.x", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
chrono = { version = "0.4", features = ["serde"] }
```

## Step 1: Data Models

Create `src/models.rs`:

```rust
use serde::{Deserialize, Serialize};
use chrono::{DateTime, Utc};

#[derive(Serialize, Deserialize, Clone, Debug)]
pub struct Post {
    pub id: u32,
    pub title: String,
    pub content: String,
    pub author: String,
    pub tags: Vec<String>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Serialize, Deserialize, Clone, Debug)]
pub struct Comment {
    pub id: u32,
    pub post_id: u32,
    pub author: String,
    pub content: String,
    pub created_at: DateTime<Utc>,
}

#[derive(Deserialize, Debug)]
pub struct CreatePost {
    pub title: String,
    pub content: String,
    pub author: String,
    pub tags: Vec<String>,
}

#[derive(Deserialize, Debug)]
pub struct UpdatePost {
    pub title: Option<String>,
    pub content: Option<String>,
    pub tags: Option<Vec<String>>,
}

#[derive(Deserialize, Debug)]
pub struct CreateComment {
    pub author: String,
    pub content: String,
}

#[derive(Deserialize, Debug)]
pub struct ListPostsQuery {
    pub tag: Option<String>,
    #[serde(default)]
    pub page: u32,
    #[serde(default = "default_limit")]
    pub limit: u32,
}

fn default_limit() -> u32 {
    10
}
```

## Step 2: Application State

Create `src/state.rs`:

```rust
use crate::models::{Post, Comment};
use std::sync::{Arc, Mutex};

#[derive(Clone)]
pub struct AppState {
    pub posts: Arc<Mutex<Vec<Post>>>,
    pub comments: Arc<Mutex<Vec<Comment>>>,
}

impl AppState {
    pub fn new() -> Self {
        Self {
            posts: Arc::new(Mutex::new(Vec::new())),
            comments: Arc::new(Mutex::new(Vec::new())),
        }
    }

    pub fn with_sample_data() -> Self {
        use chrono::Utc;

        let posts = vec![
            Post {
                id: 1,
                title: "Getting Started with Rust".to_string(),
                content: "Rust is a systems programming language...".to_string(),
                author: "Alice".to_string(),
                tags: vec!["rust".to_string(), "programming".to_string()],
                created_at: Utc::now(),
                updated_at: Utc::now(),
            },
            Post {
                id: 2,
                title: "Building Web APIs with Axum".to_string(),
                content: "Axum is a modern web framework...".to_string(),
                author: "Bob".to_string(),
                tags: vec!["rust".to_string(), "web".to_string(), "axum".to_string()],
                created_at: Utc::now(),
                updated_at: Utc::now(),
            },
        ];

        let comments = vec![
            Comment {
                id: 1,
                post_id: 1,
                author: "Charlie".to_string(),
                content: "Great article!".to_string(),
                created_at: Utc::now(),
            },
        ];

        Self {
            posts: Arc::new(Mutex::new(posts)),
            comments: Arc::new(Mutex::new(comments)),
        }
    }
}
```

## Step 3: Error Handling

Create `src/errors.rs`:

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

pub enum ApiError {
    NotFound,
    BadRequest(String),
    InternalError,
}

impl IntoResponse for ApiError {
    fn into_response(self) -> Response {
        let (status, error_code, message) = match self {
            ApiError::NotFound => (
                StatusCode::NOT_FOUND,
                "NOT_FOUND",
                "Resource not found".to_string(),
            ),
            ApiError::BadRequest(msg) => (
                StatusCode::BAD_REQUEST,
                "BAD_REQUEST",
                msg,
            ),
            ApiError::InternalError => (
                StatusCode::INTERNAL_SERVER_ERROR,
                "INTERNAL_ERROR",
                "An internal error occurred".to_string(),
            ),
        };

        (
            status,
            Json(ErrorResponse {
                error: error_code.to_string(),
                message,
            }),
        )
            .into_response()
    }
}

pub fn validate_create_post(post: &crate::models::CreatePost) -> Result<(), ApiError> {
    if post.title.trim().is_empty() {
        return Err(ApiError::BadRequest("Title cannot be empty".to_string()));
    }

    if post.content.trim().is_empty() {
        return Err(ApiError::BadRequest("Content cannot be empty".to_string()));
    }

    if post.author.trim().is_empty() {
        return Err(ApiError::BadRequest("Author cannot be empty".to_string()));
    }

    Ok(())
}

pub fn validate_create_comment(comment: &crate::models::CreateComment) -> Result<(), ApiError> {
    if comment.author.trim().is_empty() {
        return Err(ApiError::BadRequest("Author cannot be empty".to_string()));
    }

    if comment.content.trim().is_empty() {
        return Err(ApiError::BadRequest("Comment cannot be empty".to_string()));
    }

    Ok(())
}
```

## Step 4: Post Handlers

Create `src/handlers/posts.rs`:

```rust
use axum::{
    extract::{Path, Query, State},
    http::StatusCode,
    Json,
};
use chrono::Utc;

use crate::{
    models::{Post, CreatePost, UpdatePost, ListPostsQuery},
    state::AppState,
    errors::{ApiError, validate_create_post},
};

pub async fn list_posts(
    State(state): State<AppState>,
    Query(query): Query<ListPostsQuery>,
) -> Json<Vec<Post>> {
    let posts = state.posts.lock().unwrap();

    let mut filtered: Vec<Post> = posts
        .iter()
        .filter(|post| {
            if let Some(ref tag) = query.tag {
                post.tags.contains(tag)
            } else {
                true
            }
        })
        .cloned()
        .collect();

    // Sort by created_at descending
    filtered.sort_by(|a, b| b.created_at.cmp(&a.created_at));

    // Pagination
    let start = (query.page * query.limit) as usize;
    let end = start + query.limit as usize;

    let paginated = filtered.into_iter().skip(start).take(query.limit as usize).collect();

    Json(paginated)
}

pub async fn get_post(
    Path(id): Path<u32>,
    State(state): State<AppState>,
) -> Result<Json<Post>, ApiError> {
    let posts = state.posts.lock().unwrap();

    posts
        .iter()
        .find(|p| p.id == id)
        .cloned()
        .map(Json)
        .ok_or(ApiError::NotFound)
}

pub async fn create_post(
    State(state): State<AppState>,
    Json(create): Json<CreatePost>,
) -> Result<(StatusCode, Json<Post>), ApiError> {
    validate_create_post(&create)?;

    let mut posts = state.posts.lock().unwrap();

    let id = posts.iter().map(|p| p.id).max().unwrap_or(0) + 1;

    let now = Utc::now();
    let post = Post {
        id,
        title: create.title,
        content: create.content,
        author: create.author,
        tags: create.tags,
        created_at: now,
        updated_at: now,
    };

    posts.push(post.clone());

    Ok((StatusCode::CREATED, Json(post)))
}

pub async fn update_post(
    Path(id): Path<u32>,
    State(state): State<AppState>,
    Json(update): Json<UpdatePost>,
) -> Result<Json<Post>, ApiError> {
    let mut posts = state.posts.lock().unwrap();

    let post = posts
        .iter_mut()
        .find(|p| p.id == id)
        .ok_or(ApiError::NotFound)?;

    if let Some(title) = update.title {
        if title.trim().is_empty() {
            return Err(ApiError::BadRequest("Title cannot be empty".to_string()));
        }
        post.title = title;
    }

    if let Some(content) = update.content {
        if content.trim().is_empty() {
            return Err(ApiError::BadRequest("Content cannot be empty".to_string()));
        }
        post.content = content;
    }

    if let Some(tags) = update.tags {
        post.tags = tags;
    }

    post.updated_at = Utc::now();

    Ok(Json(post.clone()))
}

pub async fn delete_post(
    Path(id): Path<u32>,
    State(state): State<AppState>,
) -> Result<StatusCode, ApiError> {
    let mut posts = state.posts.lock().unwrap();

    let index = posts
        .iter()
        .position(|p| p.id == id)
        .ok_or(ApiError::NotFound)?;

    posts.remove(index);

    // Also delete associated comments
    let mut comments = state.comments.lock().unwrap();
    comments.retain(|c| c.post_id != id);

    Ok(StatusCode::NO_CONTENT)
}
```

## Step 5: Comment Handlers

Create `src/handlers/comments.rs`:

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    Json,
};
use chrono::Utc;

use crate::{
    models::{Comment, CreateComment},
    state::AppState,
    errors::{ApiError, validate_create_comment},
};

pub async fn list_comments(
    Path(post_id): Path<u32>,
    State(state): State<AppState>,
) -> Result<Json<Vec<Comment>>, ApiError> {
    // Verify post exists
    {
        let posts = state.posts.lock().unwrap();
        if !posts.iter().any(|p| p.id == post_id) {
            return Err(ApiError::NotFound);
        }
    }

    let comments = state.comments.lock().unwrap();

    let post_comments: Vec<Comment> = comments
        .iter()
        .filter(|c| c.post_id == post_id)
        .cloned()
        .collect();

    Ok(Json(post_comments))
}

pub async fn create_comment(
    Path(post_id): Path<u32>,
    State(state): State<AppState>,
    Json(create): Json<CreateComment>,
) -> Result<(StatusCode, Json<Comment>), ApiError> {
    validate_create_comment(&create)?;

    // Verify post exists
    {
        let posts = state.posts.lock().unwrap();
        if !posts.iter().any(|p| p.id == post_id) {
            return Err(ApiError::NotFound);
        }
    }

    let mut comments = state.comments.lock().unwrap();

    let id = comments.iter().map(|c| c.id).max().unwrap_or(0) + 1;

    let comment = Comment {
        id,
        post_id,
        author: create.author,
        content: create.content,
        created_at: Utc::now(),
    };

    comments.push(comment.clone());

    Ok((StatusCode::CREATED, Json(comment)))
}

pub async fn delete_comment(
    Path((post_id, comment_id)): Path<(u32, u32)>,
    State(state): State<AppState>,
) -> Result<StatusCode, ApiError> {
    let mut comments = state.comments.lock().unwrap();

    let index = comments
        .iter()
        .position(|c| c.id == comment_id && c.post_id == post_id)
        .ok_or(ApiError::NotFound)?;

    comments.remove(index);

    Ok(StatusCode::NO_CONTENT)
}
```

## Step 6: Module Organization

Create `src/handlers/mod.rs`:

```rust
pub mod posts;
pub mod comments;
```

Update `src/main.rs` to declare modules:

```rust
mod models;
mod state;
mod errors;
mod handlers;

// (rest of main.rs below)
```

## Step 7: Main Application

Update `src/main.rs`:

```rust
mod models;
mod state;
mod errors;
mod handlers;

use axum::{
    routing::{get, post, put, delete},
    Router,
};

use state::AppState;
use handlers::{posts, comments};

#[tokio::main]
async fn main() {
    // Initialize state with sample data
    let state = AppState::with_sample_data();

    // Build router
    let app = Router::new()
        // API info
        .route("/", get(api_info))
        // Posts
        .route("/posts", get(posts::list_posts).post(posts::create_post))
        .route("/posts/{id}", get(posts::get_post).put(posts::update_post).delete(posts::delete_post))
        // Comments
        .route("/posts/{post_id}/comments", get(comments::list_comments).post(comments::create_comment))
        .route("/posts/{post_id}/comments/{comment_id}", delete(comments::delete_comment))
        .with_state(state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 Blog API running on http://localhost:3000");
    println!("\n📖 Available endpoints:");
    println!("   GET    /posts");
    println!("   GET    /posts?tag=rust&page=0&limit=10");
    println!("   GET    /posts/{{id}}");
    println!("   POST   /posts");
    println!("   PUT    /posts/{{id}}");
    println!("   DELETE /posts/{{id}}");
    println!("   GET    /posts/{{post_id}}/comments");
    println!("   POST   /posts/{{post_id}}/comments");
    println!("   DELETE /posts/{{post_id}}/comments/{{comment_id}}");

    axum::serve(listener, app).await.unwrap();
}

async fn api_info() -> &'static str {
    "Blog REST API v1.0 - Built with Rust & Axum"
}
```

## Step 8: Testing the API

### List All Posts

```bash
curl http://localhost:3000/posts
```

### List Posts by Tag

```bash
curl "http://localhost:3000/posts?tag=rust"
```

### Get Single Post

```bash
curl http://localhost:3000/posts/1
```

### Create Post

```bash
curl -X POST http://localhost:3000/posts \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My New Post",
    "content": "This is the content of my new post",
    "author": "Dave",
    "tags": ["rust", "tutorial"]
  }'
```

### Update Post

```bash
curl -X PUT http://localhost:3000/posts/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Title"
  }'
```

### Delete Post

```bash
curl -X DELETE http://localhost:3000/posts/1
```

### List Comments

```bash
curl http://localhost:3000/posts/1/comments
```

### Create Comment

```bash
curl -X POST http://localhost:3000/posts/1/comments \
  -H "Content-Type: application/json" \
  -d '{
    "author": "Eve",
    "content": "Interesting post!"
  }'
```

### Delete Comment

```bash
curl -X DELETE http://localhost:3000/posts/1/comments/1
```

## Challenge Extensions

### 1. Add Search Functionality

Add `/posts/search?q=rust` endpoint that searches in title and content.

<details>
<summary>Hint</summary>

```rust
#[derive(Deserialize)]
struct SearchQuery {
    q: String,
}

async fn search_posts(
    State(state): State<AppState>,
    Query(query): Query<SearchQuery>,
) -> Json<Vec<Post>> {
    let posts = state.posts.lock().unwrap();
    let query_lower = query.q.to_lowercase();

    let results: Vec<Post> = posts
        .iter()
        .filter(|p| {
            p.title.to_lowercase().contains(&query_lower)
                || p.content.to_lowercase().contains(&query_lower)
        })
        .cloned()
        .collect();

    Json(results)
}
```
</details>

### 2. Add Statistics Endpoint

Create `GET /stats` that returns:
- Total posts
- Total comments
- Posts by tag

<details>
<summary>Hint</summary>

```rust
use std::collections::HashMap;

#[derive(Serialize)]
struct Stats {
    total_posts: usize,
    total_comments: usize,
    posts_by_tag: HashMap<String, usize>,
}

async fn get_stats(State(state): State<AppState>) -> Json<Stats> {
    let posts = state.posts.lock().unwrap();
    let comments = state.comments.lock().unwrap();

    let mut posts_by_tag = HashMap::new();
    for post in posts.iter() {
        for tag in &post.tags {
            *posts_by_tag.entry(tag.clone()).or_insert(0) += 1;
        }
    }

    Json(Stats {
        total_posts: posts.len(),
        total_comments: comments.len(),
        posts_by_tag,
    })
}
```
</details>

### 3. Add Sorting

Support `?sort=created_at&order=desc` query parameters.

### 4. Add Post Likes

- Add `likes: u32` field to Post
- Create `POST /posts/{id}/like` endpoint
- Return updated post with incremented likes

## Key Takeaways

- ✅ Organized code into modules (models, state, errors, handlers)
- ✅ Implemented complete CRUD operations
- ✅ Added input validation
- ✅ Proper error handling with custom types
- ✅ Filtering and pagination
- ✅ Nested resources (posts and comments)
- ✅ Type-safe request/response handling

## What You've Built

A production-ready REST API structure with:
- Clean code organization
- Proper error handling
- Input validation
- Filtering and pagination
- Nested resources
- Comprehensive documentation

**Next Module**: Database Integration with SQLx!

---

**Progress**: Module 12 complete! (62/90+ lessons total)
