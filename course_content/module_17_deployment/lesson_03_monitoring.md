# Module 17: Production Deployment

# Lesson 3: Monitoring & Observability

## Logging with Tracing

```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
```

```rust
use tracing::{info, warn, error};

#[tokio::main]
async fn main() {
    tracing_subscriber::fmt()
        .with_env_filter("info,my_api=debug")
        .init();

    info!("Starting server");

    // ...
}

async fn create_user(username: &str) -> Result<User, Error> {
    info!(username = %username, "Creating user");

    match create_user_in_db(username).await {
        Ok(user) => {
            info!(user_id = user.id, "User created successfully");
            Ok(user)
        }
        Err(e) => {
            error!(error = %e, "Failed to create user");
            Err(e)
        }
    }
}
```

## Metrics with Prometheus

```toml
[dependencies]
axum-prometheus = "0.6"
```

```rust
use axum_prometheus::PrometheusMetricLayer;

let (prometheus_layer, metric_handle) = PrometheusMetricLayer::pair();

let app = Router::new()
    .route("/", get(handler))
    .route("/metrics", get(|| async move { metric_handle.render() }))
    .layer(prometheus_layer);
```

## Error Tracking (Sentry)

```toml
[dependencies]
sentry = "0.32"
```

```rust
let _guard = sentry::init(("YOUR_SENTRY_DSN", sentry::ClientOptions {
    release: sentry::release_name!(),
    ..Default::default()
}));

// Errors automatically tracked
sentry::capture_error(&error);
```

## Key Takeaways

- ✅ Use tracing for structured logging
- ✅ Expose Prometheus metrics
- ✅ Integrate error tracking (Sentry)
- ✅ Monitor in production

**Module 17 Complete!**

---

**Progress**: Module 17 complete! (81/90+ lessons total)
