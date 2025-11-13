# Module 12: Web Development with Rust

# Lesson 3: Routing & Extractors

## Path Parameters

Often, you need dynamic URLs like `/users/123` or `/posts/my-first-post`.

### Basic Path Parameters

```rust
use axum::{
    routing::get,
    Router,
    extract::Path,
};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/users/{id}", get(get_user))
        .route("/posts/{slug}", get(get_post));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}

// Extract a single parameter
async fn get_user(Path(id): Path<u32>) -> String {
    format!("Getting user with ID: {}", id)
}

async fn get_post(Path(slug): Path<String>) -> String {
    format!("Getting post with slug: {}", slug)
}
```

**Test it:**

```bash
curl http://localhost:3000/users/42
# Output: Getting user with ID: 42

curl http://localhost:3000/posts/hello-world
# Output: Getting post with slug: hello-world
```

### Important: Path Syntax Change

Axum 0.8 changed path parameter syntax:

✅ **Axum 0.8+ (new syntax):**
```rust
.route("/users/{id}", get(handler))        // Single parameter
.route("/files/{*path}", get(handler))     // Catch-all
```

❌ **Axum 0.7 and earlier (old syntax):**
```rust
.route("/users/:id", get(handler))         // Old style
.route("/files/*path", get(handler))       // Old style
```

**Use the new `{param}` syntax!**

## Multiple Path Parameters

```rust
use axum::{
    routing::get,
    Router,
    extract::Path,
};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/users/{user_id}/posts/{post_id}", get(get_user_post));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}

// Method 1: Tuple extraction
async fn get_user_post(Path((user_id, post_id)): Path<(u32, u32)>) -> String {
    format!("User {} - Post {}", user_id, post_id)
}
```

**Test:**
```bash
curl http://localhost:3000/users/5/posts/42
# Output: User 5 - Post 42
```

### Using a Struct for Parameters

For better readability with many parameters:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct PostPath {
    user_id: u32,
    post_id: u32,
}

async fn get_user_post(Path(params): Path<PostPath>) -> String {
    format!("User {} - Post {}", params.user_id, params.post_id)
}
```

## Query Parameters

Query parameters come after `?` in the URL: `/search?q=rust&limit=10`

```rust
use axum::extract::Query;
use serde::Deserialize;

#[derive(Deserialize)]
struct SearchParams {
    q: String,
    #[serde(default = "default_limit")]
    limit: u32,
}

fn default_limit() -> u32 {
    10
}

async fn search(Query(params): Query<SearchParams>) -> String {
    format!("Searching for '{}' with limit {}", params.q, params.limit)
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/search", get(search));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```

**Test:**

```bash
curl "http://localhost:3000/search?q=rust"
# Output: Searching for 'rust' with limit 10

curl "http://localhost:3000/search?q=rust&limit=20"
# Output: Searching for 'rust' with limit 20
```

### Optional Query Parameters

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct Pagination {
    page: Option<u32>,
    per_page: Option<u32>,
}

async fn list_items(Query(pagination): Query<Pagination>) -> String {
    let page = pagination.page.unwrap_or(1);
    let per_page = pagination.per_page.unwrap_or(20);

    format!("Page {} with {} items per page", page, per_page)
}
```

## What are Extractors?

**Extractors** pull data out of HTTP requests in a type-safe way.

### Common Extractors

| Extractor | What it extracts |
|-----------|------------------|
| `Path<T>` | URL path parameters |
| `Query<T>` | Query string parameters |
| `Json<T>` | JSON request body |
| `Form<T>` | Form data |
| `Extension<T>` | Shared state |
| `headers::HeaderMap` | HTTP headers |

## Combining Multiple Extractors

You can use multiple extractors in the same handler:

```rust
use axum::{
    extract::{Path, Query},
    Json,
};
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct UserQuery {
    include_posts: bool,
}

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
    posts: Option<Vec<String>>,
}

async fn get_user_detailed(
    Path(id): Path<u32>,
    Query(query): Query<UserQuery>,
) -> Json<User> {
    let posts = if query.include_posts {
        Some(vec!["Post 1".to_string(), "Post 2".to_string()])
    } else {
        None
    };

    Json(User {
        id,
        name: format!("User {}", id),
        posts,
    })
}
```

**Test:**
```bash
curl "http://localhost:3000/users/5?include_posts=true"
# Output: {"id":5,"name":"User 5","posts":["Post 1","Post 2"]}
```

## Catch-All Routes

Capture the rest of the path:

```rust
async fn serve_file(Path(path): Path<String>) -> String {
    format!("Serving file: {}", path)
}

let app = Router::new()
    .route("/files/{*path}", get(serve_file));
```

**Test:**
```bash
curl http://localhost:3000/files/docs/readme.md
# Output: Serving file: docs/readme.md

curl http://localhost:3000/files/a/b/c/d.txt
# Output: Serving file: a/b/c/d.txt
```

## Nested Routers

Organize large applications by nesting routers:

```rust
use axum::Router;
use axum::routing::get;

#[tokio::main]
async fn main() {
    // User routes
    let user_routes = Router::new()
        .route("/", get(list_users))
        .route("/{id}", get(get_user));

    // Post routes
    let post_routes = Router::new()
        .route("/", get(list_posts))
        .route("/{id}", get(get_post));

    // Main app
    let app = Router::new()
        .route("/", get(home))
        .nest("/users", user_routes)
        .nest("/posts", post_routes);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 Server running!");
    println!("   GET /");
    println!("   GET /users");
    println!("   GET /users/{{id}}");
    println!("   GET /posts");
    println!("   GET /posts/{{id}}");

    axum::serve(listener, app).await.unwrap();
}

async fn home() -> &'static str {
    "API Home"
}

async fn list_users() -> &'static str {
    "List of all users"
}

async fn get_user(Path(id): Path<u32>) -> String {
    format!("User {}", id)
}

async fn list_posts() -> &'static str {
    "List of all posts"
}

async fn get_post(Path(id): Path<u32>) -> String {
    format!("Post {}", id)
}
```

## Router Fallback (404 Handler)

Handle routes that don't match:

```rust
use axum::http::StatusCode;

async fn not_found() -> (StatusCode, &'static str) {
    (StatusCode::NOT_FOUND, "404 - Page not found")
}

let app = Router::new()
    .route("/", get(home))
    .route("/about", get(about))
    .fallback(not_found);
```

**Test:**
```bash
curl http://localhost:3000/nonexistent
# Output: 404 - Page not found
```

## Complete Example: Blog API Routing

```rust
use axum::{
    routing::{get, post, put, delete},
    Router,
    extract::{Path, Query},
    Json,
    http::StatusCode,
};
use serde::{Deserialize, Serialize};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(api_info))
        .route("/posts", get(list_posts).post(create_post))
        .route("/posts/{id}", get(get_post).put(update_post).delete(delete_post))
        .route("/posts/{id}/comments", get(list_comments))
        .fallback(not_found);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 Blog API running on http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}

async fn api_info() -> &'static str {
    "Blog API v1.0"
}

#[derive(Deserialize)]
struct ListQuery {
    #[serde(default)]
    page: u32,
    #[serde(default = "default_per_page")]
    per_page: u32,
}

fn default_per_page() -> u32 {
    10
}

async fn list_posts(Query(query): Query<ListQuery>) -> String {
    format!("Listing posts - page {}, {} per page", query.page, query.per_page)
}

#[derive(Deserialize)]
struct CreatePost {
    title: String,
    content: String,
}

async fn create_post(Json(post): Json<CreatePost>) -> (StatusCode, String) {
    (
        StatusCode::CREATED,
        format!("Created post: {}", post.title)
    )
}

async fn get_post(Path(id): Path<u32>) -> String {
    format!("Getting post {}", id)
}

#[derive(Deserialize)]
struct UpdatePost {
    title: Option<String>,
    content: Option<String>,
}

async fn update_post(
    Path(id): Path<u32>,
    Json(update): Json<UpdatePost>,
) -> String {
    format!("Updating post {}", id)
}

async fn delete_post(Path(id): Path<u32>) -> (StatusCode, String) {
    (StatusCode::NO_CONTENT, format!("Deleted post {}", id))
}

async fn list_comments(Path(post_id): Path<u32>) -> String {
    format!("Listing comments for post {}", post_id)
}

async fn not_found() -> (StatusCode, &'static str) {
    (StatusCode::NOT_FOUND, "Endpoint not found")
}
```

## Practice Exercises

### Exercise 1: E-commerce Routes

Build routing for an e-commerce API:

**Requirements:**
- `/products` - GET (list products), POST (create product)
- `/products/{id}` - GET (get product), PUT (update), DELETE (delete)
- `/categories` - GET (list categories)
- `/products?category=electronics&sort=price` - Filter and sort
- `/orders/{order_id}/items` - GET items in an order

<details>
<summary>Solution</summary>

```rust
use axum::{
    routing::{get, post, put, delete},
    Router,
    extract::{Path, Query},
};
use serde::Deserialize;

#[derive(Deserialize)]
struct ProductQuery {
    category: Option<String>,
    sort: Option<String>,
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/products", get(list_products).post(create_product))
        .route("/products/{id}", get(get_product).put(update_product).delete(delete_product))
        .route("/categories", get(list_categories))
        .route("/orders/{order_id}/items", get(list_order_items));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 E-commerce API running!");
    axum::serve(listener, app).await.unwrap();
}

async fn list_products(Query(query): Query<ProductQuery>) -> String {
    format!(
        "Products - category: {:?}, sort: {:?}",
        query.category, query.sort
    )
}

async fn create_product() -> &'static str {
    "Create product"
}

async fn get_product(Path(id): Path<u32>) -> String {
    format!("Get product {}", id)
}

async fn update_product(Path(id): Path<u32>) -> String {
    format!("Update product {}", id)
}

async fn delete_product(Path(id): Path<u32>) -> String {
    format!("Delete product {}", id)
}

async fn list_categories() -> &'static str {
    "List categories"
}

async fn list_order_items(Path(order_id): Path<u32>) -> String {
    format!("List items for order {}", order_id)
}
```
</details>

### Exercise 2: File Server with Catch-All

Create a file server that:
- `/files/{*path}` - serves any file path
- Extracts the full path
- Returns "Serving: [path]"

<details>
<summary>Solution</summary>

```rust
use axum::{
    routing::get,
    Router,
    extract::Path,
};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(home))
        .route("/files/{*path}", get(serve_file));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("🚀 File server running!");
    axum::serve(listener, app).await.unwrap();
}

async fn home() -> &'static str {
    "File Server - use /files/path/to/file"
}

async fn serve_file(Path(path): Path<String>) -> String {
    format!("Serving: {}", path)
}
```

**Test:**
```bash
curl http://localhost:3000/files/documents/report.pdf
# Output: Serving: documents/report.pdf
```
</details>

### Exercise 3: Search API with Multiple Parameters

Create `/search` endpoint that accepts:
- `q` - search query (required)
- `category` - filter by category (optional)
- `page` - page number (optional, default 1)
- `limit` - results per page (optional, default 20)

<details>
<summary>Solution</summary>

```rust
use axum::{routing::get, Router, extract::Query};
use serde::Deserialize;

#[derive(Deserialize)]
struct SearchParams {
    q: String,
    category: Option<String>,
    #[serde(default = "default_page")]
    page: u32,
    #[serde(default = "default_limit")]
    limit: u32,
}

fn default_page() -> u32 {
    1
}

fn default_limit() -> u32 {
    20
}

async fn search(Query(params): Query<SearchParams>) -> String {
    format!(
        "Search: '{}' in category: {:?}, page: {}, limit: {}",
        params.q, params.category, params.page, params.limit
    )
}

#[tokio::main]
async fn main() {
    let app = Router::new().route("/search", get(search));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```

**Test:**
```bash
curl "http://localhost:3000/search?q=rust"
curl "http://localhost:3000/search?q=rust&category=books&page=2&limit=10"
```
</details>

## Common Patterns

### Versioned APIs

```rust
let v1_routes = Router::new()
    .route("/users", get(v1_users));

let v2_routes = Router::new()
    .route("/users", get(v2_users));

let app = Router::new()
    .nest("/api/v1", v1_routes)
    .nest("/api/v2", v2_routes);
```

### RESTful Resource Routes

```rust
fn resource_routes<T>() -> Router {
    Router::new()
        .route("/", get(list::<T>).post(create::<T>))
        .route("/{id}", get(get::<T>).put(update::<T>).delete(delete::<T>))
}
```

## Key Takeaways

- ✅ Path parameters: `/users/{id}` extracts dynamic segments
- ✅ Query parameters: `/search?q=rust` extracts URL parameters
- ✅ Extractors pull typed data from requests
- ✅ Can combine multiple extractors in one handler
- ✅ Catch-all routes: `{*path}` captures remaining path
- ✅ Nested routers organize code into modules
- ✅ Fallback handler catches 404s

**Next**: JSON APIs and working with request/response bodies!

---

**Progress**: Module 12, Lesson 3 complete (60/90+ lessons total)
