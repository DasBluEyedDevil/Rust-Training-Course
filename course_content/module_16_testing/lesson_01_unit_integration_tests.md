# Module 16: Testing & Quality

# Lesson 1: Unit & Integration Testing

## Unit Tests

Test individual functions:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 2), 4);
    }

    #[test]
    fn test_user_validation() {
        assert!(validate_email("test@example.com"));
        assert!(!validate_email("invalid"));
    }
}
```

## Testing Database Code

```toml
[dev-dependencies]
sqlx = { version = "0.8.x", features = ["runtime-tokio", "postgres"] }
```

```rust
#[sqlx::test]
async fn test_create_user(pool: PgPool) -> sqlx::Result<()> {
    let user = create_user(&pool, "testuser", "test@example.com").await?;
    assert_eq!(user.username, "testuser");
    Ok(())
}
```

## Testing API Endpoints

```toml
[dev-dependencies]
axum-test = "14"
```

```rust
use axum_test::TestServer;

#[tokio::test]
async fn test_list_posts() {
    let app = create_app();
    let server = TestServer::new(app).unwrap();

    let response = server.get("/posts").await;
    response.assert_status_ok();
    response.assert_json(&vec![/* expected posts */]);
}

#[tokio::test]
async fn test_auth_required() {
    let app = create_app();
    let server = TestServer::new(app).unwrap();

    let response = server.post("/posts")
        .json(&json!({"title": "Test"}))
        .await;

    response.assert_status(StatusCode::UNAUTHORIZED);
}
```

## Key Takeaways

- ✅ Unit tests for functions
- ✅ `#[sqlx::test]` for database tests
- ✅ axum-test for API testing
- ✅ Test authentication and authorization

**Next**: API documentation and code quality!

---

**Progress**: Module 16, Lesson 1 complete (76/90+ lessons total)
