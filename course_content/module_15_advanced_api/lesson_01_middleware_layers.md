# Module 15: Advanced API Development

# Lesson 1: Middleware & Request/Response Processing

## What is Middleware?

**Middleware** processes requests before they reach handlers and responses before they reach clients.

### Tower Layer System

Axum uses **Tower** for middleware. Middleware can:
- Log requests
- Add headers
- Authenticate users
- Transform requests/responses
- Handle errors

## Built-in Middleware

### Logging

```toml
[dependencies]
tower-http = { version = "0.5", features = ["trace"] }
tracing = "0.1"
tracing-subscriber = "0.3"
```

```rust
use tower_http::trace::TraceLayer;
use tracing_subscriber;

#[tokio::main]
async fn main() {
    tracing_subscriber::fmt::init();

    let app = Router::new()
        .route("/", get(handler))
        .layer(TraceLayer::new_for_http());

    // Logs every request!
}
```

### CORS

```rust
use tower_http::cors::{CorsLayer, Any};

let cors = CorsLayer::new()
    .allow_origin(Any)
    .allow_methods(Any)
    .allow_headers(Any);

let app = Router::new()
    .route("/api/data", get(get_data))
    .layer(cors);
```

### Compression

```rust
use tower_http::compression::CompressionLayer;

let app = Router::new()
    .route("/api/data", get(get_large_data))
    .layer(CompressionLayer::new());  // Gzip compression
```

## Custom Middleware

### Request ID Middleware

```rust
use axum::{
    extract::Request,
    middleware::Next,
    response::Response,
    http::header,
};
use uuid::Uuid;

async fn request_id_middleware(
    mut req: Request,
    next: Next,
) -> Response {
    let request_id = Uuid::new_v4().to_string();

    // Add to request
    req.extensions_mut().insert(request_id.clone());

    // Process request
    let mut response = next.run(req).await;

    // Add to response headers
    response.headers_mut().insert(
        "X-Request-ID",
        request_id.parse().unwrap(),
    );

    response
}

// Use it
let app = Router::new()
    .route("/", get(handler))
    .layer(middleware::from_fn(request_id_middleware));
```

### Timing Middleware

```rust
use std::time::Instant;

async fn timing_middleware(req: Request, next: Next) -> Response {
    let start = Instant::now();
    let path = req.uri().path().to_owned();

    let response = next.run(req).await;

    let duration = start.elapsed();
    println!("{} took {:?}", path, duration);

    response
}
```

## State in Middleware

```rust
#[derive(Clone)]
struct AppState {
    db: PgPool,
    config: Config,
}

async fn db_middleware(
    State(state): State<AppState>,
    req: Request,
    next: Next,
) -> Response {
    // Use state.db in middleware
    next.run(req).await
}

let app = Router::new()
    .route("/", get(handler))
    .layer(middleware::from_fn_with_state(state.clone(), db_middleware))
    .with_state(state);
```

## Response Interceptors

```rust
async fn add_server_header(
    req: Request,
    next: Next,
) -> Response {
    let mut response = next.run(req).await;

    response.headers_mut().insert(
        "Server",
        "MyAPI/1.0".parse().unwrap(),
    );

    response
}
```

## Error Handling Middleware

```rust
async fn error_handler(
    req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    match next.run(req).await.status() {
        status if status.is_server_error() => {
            println!("Server error occurred");
            Err(StatusCode::INTERNAL_SERVER_ERROR)
        }
        _ => Ok(next.run(req).await),
    }
}
```

## Layer Ordering

Layers wrap in reverse order:

```rust
let app = Router::new()
    .route("/", get(handler))
    .layer(layer3)  // Runs third (outermost)
    .layer(layer2)  // Runs second
    .layer(layer1); // Runs first (innermost)

// Request flow: layer3 → layer2 → layer1 → handler → layer1 → layer2 → layer3
```

## Key Takeaways

- ✅ Middleware processes requests/responses
- ✅ Tower provides composable layers
- ✅ Built-in: logging, CORS, compression
- ✅ Custom middleware with `from_fn`
- ✅ Access state in middleware
- ✅ Layer ordering matters!

**Next**: API versioning and documentation!

---

**Progress**: Module 15, Lesson 1 complete (73/90+ lessons total)
