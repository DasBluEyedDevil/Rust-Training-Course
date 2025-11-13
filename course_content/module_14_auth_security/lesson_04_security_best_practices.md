# Module 14: Authentication & Security

# Lesson 4: Security Best Practices & Common Vulnerabilities

## OWASP Top 10 for Web Applications

The **OWASP Top 10** lists the most critical security risks.

### 1. Broken Access Control

**Problem:** Users can access resources they shouldn't.

❌ **Vulnerable:**
```rust
async fn delete_post(
    Path(post_id): Path<i32>,
) -> StatusCode {
    // Anyone can delete any post!
    sqlx::query!("DELETE FROM posts WHERE id = $1", post_id)
        .execute(&pool)
        .await?;
    StatusCode::NO_CONTENT
}
```

✅ **Secure:**
```rust
async fn delete_post(
    State(pool): State<PgPool>,
    user: CurrentUser,
    Path(post_id): Path<i32>,
) -> Result<StatusCode, ApiError> {
    // Get post ownership
    let post = sqlx::query!("SELECT user_id FROM posts WHERE id = $1", post_id)
        .fetch_one(&pool)
        .await?;

    // Check authorization
    if post.user_id != user.user_id && !user.is_admin() {
        return Err(ApiError::Forbidden("Not authorized".into()));
    }

    sqlx::query!("DELETE FROM posts WHERE id = $1", post_id)
        .execute(&pool)
        .await?;

    Ok(StatusCode::NO_CONTENT)
}
```

### 2. SQL Injection

**Problem:** User input executed as SQL code.

❌ **Vulnerable:**
```rust
async fn search_users(query: String) -> Vec<User> {
    // NEVER DO THIS!
    let sql = format!("SELECT * FROM users WHERE name = '{}'", query);
    sqlx::query(&sql).fetch_all(&pool).await.unwrap()
}

// Attacker sends: "'; DROP TABLE users; --"
// Executes: SELECT * FROM users WHERE name = ''; DROP TABLE users; --'
```

✅ **Secure:**
```rust
async fn search_users(query: String) -> Vec<User> {
    // Use parameterized queries
    sqlx::query_as!(
        User,
        "SELECT * FROM users WHERE name LIKE $1",
        format!("%{}%", query)
    )
    .fetch_all(&pool)
    .await
    .unwrap()
}
```

**Rust's type system helps!** `query!` and `query_as!` macros prevent SQL injection.

### 3. Cryptographic Failures

**Problem:** Weak encryption, storing sensitive data unencrypted.

✅ **Best Practices:**
- Use `argon2` for passwords (never MD5 or SHA1)
- Use HTTPS in production
- Store secrets in environment variables
- Use strong JWT secrets (32+ characters)

```rust
// Generate strong secret:
// openssl rand -base64 32

let jwt_secret = std::env::var("JWT_SECRET")
    .expect("JWT_SECRET must be set");
```

### 4. Insecure Design

**Problem:** Flawed architecture from the start.

✅ **Secure by Design:**
- Default deny (require explicit authorization)
- Principle of least privilege
- Defense in depth (multiple security layers)
- Fail securely (errors don't reveal info)

### 5. Security Misconfiguration

**Problem:** Leaving default settings, exposing debug info.

❌ **Vulnerable:**
```rust
// Exposing internal error details
return Err(format!("Database error: {}", err));  // Shows SQL error to user!
```

✅ **Secure:**
```rust
// Generic error message to user
return Err(ApiError::InternalError("An error occurred".into()));

// Log detailed error server-side
eprintln!("Database error: {}", err);
```

**Production checklist:**
- [ ] Change default passwords
- [ ] Disable debug mode
- [ ] Remove development endpoints
- [ ] Set up proper CORS
- [ ] Use environment variables for config

### 6. Vulnerable and Outdated Components

**Problem:** Using libraries with known vulnerabilities.

✅ **Prevention:**
```bash
# Check for vulnerabilities
cargo audit

# Update dependencies
cargo update
```

### 7. Identification and Authentication Failures

**Problem:** Weak authentication mechanisms.

✅ **Best Practices:**
- Enforce strong passwords
- Implement rate limiting
- Use multi-factor authentication (MFA)
- Secure session management
- Prevent brute-force attacks

```rust
// Password requirements
fn validate_password(password: &str) -> Result<(), String> {
    if password.len() < 12 {
        return Err("Password must be at least 12 characters".into());
    }
    if !password.chars().any(|c| c.is_uppercase()) {
        return Err("Password must contain uppercase letter".into());
    }
    if !password.chars().any(|c| c.is_lowercase()) {
        return Err("Password must contain lowercase letter".into());
    }
    if !password.chars().any(|c| c.is_numeric()) {
        return Err("Password must contain number".into());
    }
    if !password.chars().any(|c| "!@#$%^&*".contains(c)) {
        return Err("Password must contain special character".into());
    }
    Ok(())
}
```

### 8. Software and Data Integrity Failures

**Problem:** Accepting untrusted data without validation.

✅ **Input Validation:**
```rust
#[derive(Deserialize)]
struct CreatePost {
    #[serde(deserialize_with = "validate_title")]
    title: String,
    content: String,
}

fn validate_title<'de, D>(deserializer: D) -> Result<String, D::Error>
where
    D: serde::Deserializer<'de>,
{
    let title = String::deserialize(deserializer)?;

    if title.trim().is_empty() {
        return Err(serde::de::Error::custom("Title cannot be empty"));
    }

    if title.len() > 255 {
        return Err(serde::de::Error::custom("Title too long"));
    }

    // Sanitize HTML/script tags
    if title.contains('<') || title.contains('>') {
        return Err(serde::de::Error::custom("Title cannot contain HTML"));
    }

    Ok(title)
}
```

### 9. Security Logging and Monitoring Failures

**Problem:** Not logging security events.

✅ **Audit Logging:**
```rust
async fn log_security_event(
    pool: &PgPool,
    event_type: &str,
    user_id: Option<i32>,
    ip_address: &str,
    details: Option<serde_json::Value>,
) -> Result<(), sqlx::Error> {
    sqlx::query!(
        r#"
        INSERT INTO security_log (event_type, user_id, ip_address, details)
        VALUES ($1, $2, $3, $4)
        "#,
        event_type,
        user_id,
        ip_address,
        details
    )
    .execute(pool)
    .await?;

    Ok(())
}

// Log failed login attempts
async fn login(req: LoginRequest) -> Result<LoginResponse, ApiError> {
    match authenticate(&req.username, &req.password).await {
        Ok(user) => {
            log_security_event(
                &pool,
                "login_success",
                Some(user.id),
                &ip_address,
                None,
            ).await?;
            Ok(user)
        }
        Err(e) => {
            log_security_event(
                &pool,
                "login_failed",
                None,
                &ip_address,
                Some(json!({"username": req.username})),
            ).await?;
            Err(e)
        }
    }
}
```

### 10. Server-Side Request Forgery (SSRF)

**Problem:** Making requests to internal services based on user input.

❌ **Vulnerable:**
```rust
async fn fetch_url(url: String) -> String {
    // User could access internal services!
    // url = "http://localhost:8080/admin/delete-everything"
    reqwest::get(&url).await.unwrap().text().await.unwrap()
}
```

✅ **Secure:**
```rust
async fn fetch_url(url: String) -> Result<String, ApiError> {
    // Validate URL
    let parsed = url::Url::parse(&url)
        .map_err(|_| ApiError::BadRequest("Invalid URL".into()))?;

    // Whitelist allowed hosts
    let allowed_hosts = ["api.example.com", "cdn.example.com"];
    if !allowed_hosts.contains(&parsed.host_str().unwrap_or("")) {
        return Err(ApiError::BadRequest("Host not allowed".into()));
    }

    // Blacklist private IPs
    if parsed.host_str().unwrap_or("").starts_with("192.168.")
        || parsed.host_str().unwrap_or("").starts_with("10.")
        || parsed.host_str().unwrap_or("") == "localhost"
    {
        return Err(ApiError::BadRequest("Cannot access private networks".into()));
    }

    let response = reqwest::get(url).await?.text().await?;
    Ok(response)
}
```

## Cross-Site Scripting (XSS)

**Problem:** Injecting malicious scripts.

✅ **Prevention:**
- Sanitize user input
- Escape output
- Use Content Security Policy headers

```rust
use ammonia::clean;

fn sanitize_html(input: &str) -> String {
    // Strip all HTML tags
    clean(input)
}

async fn create_post(Json(req): Json<CreatePost>) -> Result<Json<Post>, ApiError> {
    let sanitized_content = sanitize_html(&req.content);

    let post = sqlx::query_as!(
        Post,
        "INSERT INTO posts (title, content) VALUES ($1, $2) RETURNING *",
        req.title,
        sanitized_content
    )
    .fetch_one(&pool)
    .await?;

    Ok(Json(post))
}
```

## Cross-Site Request Forgery (CSRF)

**Problem:** Forcing users to execute unwanted actions.

✅ **Prevention with SameSite cookies:**
```rust
use axum::http::header::{SET_COOKIE, HeaderValue};

// Set cookie with SameSite=Strict
let cookie = format!(
    "session={}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600",
    session_id
);

let mut headers = HeaderMap::new();
headers.insert(SET_COOKIE, HeaderValue::from_str(&cookie)?);
```

## CORS (Cross-Origin Resource Sharing)

Control which domains can access your API:

```toml
[dependencies]
tower-http = { version = "0.5", features = ["cors"] }
```

```rust
use tower_http::cors::{CorsLayer, Any};
use axum::http::Method;

let cors = CorsLayer::new()
    .allow_origin("https://yourdomain.com".parse::<HeaderValue>()?)
    .allow_methods([Method::GET, Method::POST, Method::PUT, Method::DELETE])
    .allow_headers(Any)
    .allow_credentials(true);

let app = Router::new()
    .route("/api/posts", get(list_posts))
    .layer(cors);
```

## Rate Limiting

Prevent abuse and DDoS:

```toml
[dependencies]
tower-governor = "0.3"
```

```rust
use tower_governor::{
    governor::GovernorConfigBuilder,
    GovernorLayer,
};

// Allow 100 requests per minute
let governor_conf = Box::new(
    GovernorConfigBuilder::default()
        .per_second(2)
        .burst_size(5)
        .finish()
        .unwrap(),
);

let app = Router::new()
    .route("/api/posts", get(list_posts))
    .layer(GovernorLayer {
        config: Box::leak(governor_conf),
    });
```

## Environment Variables Best Practices

```rust
use dotenvy::dotenv;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Load .env only in development
    #[cfg(debug_assertions)]
    dotenv().ok();

    // Fail fast if required vars missing
    let database_url = std::env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");

    let jwt_secret = std::env::var("JWT_SECRET")
        .expect("JWT_SECRET must be set");

    // Validate secret strength
    if jwt_secret.len() < 32 {
        panic!("JWT_SECRET must be at least 32 characters");
    }

    // ...
}
```

**.env.example** (commit this):
```env
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
JWT_SECRET=your-secret-key-change-in-production
RUST_LOG=info
```

**.env** (never commit this):
```env
DATABASE_URL=postgresql://prod_user:actual_password@db.example.com:5432/prod_db
JWT_SECRET=8f3d7e9a2c1b4d6f8e7a9c2b1d4f6e8a7c9b2d1f4e6a8c7b9d2e1f4a6c8e7b9d
RUST_LOG=warn
```

## Production Security Checklist

### Before Deployment

- [ ] All secrets in environment variables
- [ ] Strong password requirements enforced
- [ ] Rate limiting enabled
- [ ] CORS configured properly
- [ ] HTTPS enforced (no HTTP)
- [ ] SQL injection prevented (use `query!`)
- [ ] Input validation on all endpoints
- [ ] Error messages don't leak information
- [ ] Audit logging enabled
- [ ] Dependencies updated (`cargo audit`)
- [ ] No debug/test endpoints in production
- [ ] JWT secrets are strong (32+ chars)
- [ ] Password hashing with argon2
- [ ] CSRF protection enabled
- [ ] XSS prevention (sanitize HTML)
- [ ] Role-based access control implemented

### Monitoring

- [ ] Log all authentication events
- [ ] Monitor for failed login attempts
- [ ] Alert on suspicious activity
- [ ] Track API usage patterns
- [ ] Database query performance monitoring

### Incident Response

- [ ] Backup and recovery procedures
- [ ] Rollback plan
- [ ] Security incident response plan
- [ ] Contact information for security team

## Key Takeaways

- ✅ Follow OWASP Top 10 guidelines
- ✅ **Never trust user input** - always validate and sanitize
- ✅ Use parameterized queries (prevents SQL injection)
- ✅ Hash passwords with argon2
- ✅ Implement proper authorization checks
- ✅ Use HTTPS in production
- ✅ Store secrets in environment variables
- ✅ Enable rate limiting
- ✅ Configure CORS properly
- ✅ Log security events
- ✅ Keep dependencies updated
- ✅ Sanitize HTML to prevent XSS
- ✅ Generic error messages (don't leak info)

**Next**: Complete authentication system practice project!

---

**Progress**: Module 14, Lesson 4 complete (71/90+ lessons total)
