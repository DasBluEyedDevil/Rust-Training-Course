# Module 15: Advanced API Development

# Lesson 2: API Versioning & Documentation

## API Versioning

Handle breaking changes gracefully.

### URL Path Versioning

```rust
let v1_routes = Router::new()
    .route("/users", get(v1::list_users));

let v2_routes = Router::new()
    .route("/users", get(v2::list_users));

let app = Router::new()
    .nest("/api/v1", v1_routes)
    .nest("/api/v2", v2_routes);
```

### Header Versioning

```rust
async fn version_middleware(req: Request, next: Next) -> Result<Response, StatusCode> {
    let version = req
        .headers()
        .get("API-Version")
        .and_then(|v| v.to_str().ok())
        .unwrap_or("v1");

    req.extensions_mut().insert(version.to_string());
    Ok(next.run(req).await)
}
```

## OpenAPI Documentation

```toml
[dependencies]
utoipa = { version = "4", features = ["axum_extras"] }
utoipa-swagger-ui = { version = "6", features = ["axum"] }
```

```rust
use utoipa::{OpenApi, ToSchema};
use utoipa_swagger_ui::SwaggerUi;

#[derive(ToSchema, Serialize)]
struct User {
    id: i32,
    username: String,
}

#[utoipa::path(
    get,
    path = "/users",
    responses(
        (status = 200, description = "List all users", body = Vec<User>)
    )
)]
async fn list_users() -> Json<Vec<User>> {
    // ...
}

#[derive(OpenApi)]
#[openapi(
    paths(list_users),
    components(schemas(User))
)]
struct ApiDoc;

let app = Router::new()
    .merge(SwaggerUi::new("/swagger-ui")
        .url("/api-docs/openapi.json", ApiDoc::openapi()));
```

Visit `http://localhost:3000/swagger-ui` for interactive docs!

## Key Takeaways

- ✅ Version APIs to handle breaking changes
- ✅ Use URL paths or headers for versions
- ✅ Auto-generate OpenAPI docs
- ✅ Swagger UI for interactive testing

**Next**: WebSockets and real-time features!

---

**Progress**: Module 15, Lesson 2 complete (74/90+ lessons total)
