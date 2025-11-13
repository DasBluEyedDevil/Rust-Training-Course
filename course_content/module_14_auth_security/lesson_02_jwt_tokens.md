# Module 14: Authentication & Security

# Lesson 2: JWT Tokens for Stateless Authentication

## What are JWTs?

**JWT (JSON Web Token)** is a compact, URL-safe token format for authentication.

**Structure:**
```
header.payload.signature
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Three parts:**
1. **Header** - Algorithm and token type
2. **Payload** - Claims (user data)
3. **Signature** - Verifies token hasn't been tampered with

## JWT vs Sessions

### Sessions (Stateful)
- Server stores session data in database/memory
- Client receives session ID cookie
- Server looks up session on each request
- ❌ Doesn't scale well (requires shared state)
- ✅ Easy to invalidate

### JWT (Stateless)
- Server creates signed token with user data
- Client stores token (localStorage/cookie)
- Client sends token with each request
- Server verifies signature (no database lookup)
- ✅ Scales horizontally (no shared state needed)
- ❌ Hard to invalidate before expiration

## Setup

```toml
[dependencies]
jsonwebtoken = "9"
serde = { version = "1.0", features = ["derive"] }
chrono = "0.4"
```

## Creating JWTs

```rust
use jsonwebtoken::{encode, decode, Header, Validation, EncodingKey, DecodingKey};
use serde::{Deserialize, Serialize};
use chrono::{Utc, Duration};

#[derive(Debug, Serialize, Deserialize)]
struct Claims {
    sub: String,      // Subject (user ID)
    exp: usize,       // Expiration time
    iat: usize,       // Issued at
    user_id: i32,
    username: String,
}

fn create_jwt(user_id: i32, username: String, secret: &str) -> Result<String, jsonwebtoken::errors::Error> {
    let now = Utc::now();
    let expiration = now + Duration::hours(24);

    let claims = Claims {
        sub: user_id.to_string(),
        exp: expiration.timestamp() as usize,
        iat: now.timestamp() as usize,
        user_id,
        username,
    };

    encode(
        &Header::default(),
        &claims,
        &EncodingKey::from_secret(secret.as_bytes())
    )
}

fn main() {
    let secret = "my-secret-key-keep-this-safe";
    let token = create_jwt(1, "alice".to_string(), secret).unwrap();

    println!("JWT Token: {}", token);
}
```

## Verifying JWTs

```rust
fn verify_jwt(token: &str, secret: &str) -> Result<Claims, jsonwebtoken::errors::Error> {
    let token_data = decode::<Claims>(
        token,
        &DecodingKey::from_secret(secret.as_bytes()),
        &Validation::default()
    )?;

    Ok(token_data.claims)
}

fn main() {
    let secret = "my-secret-key";
    let token = create_jwt(1, "alice".to_string(), secret).unwrap();

    match verify_jwt(&token, secret) {
        Ok(claims) => {
            println!("Valid token!");
            println!("User ID: {}", claims.user_id);
            println!("Username: {}", claims.username);
        }
        Err(e) => println!("Invalid token: {}", e),
    }
}
```

## Login with JWT

```rust
use axum::{
    extract::State,
    http::StatusCode,
    Json,
};
use serde::{Deserialize, Serialize};
use sqlx::PgPool;

#[derive(Deserialize)]
struct LoginRequest {
    username: String,
    password: String,
}

#[derive(Serialize)]
struct LoginResponse {
    token: String,
    user: UserResponse,
}

#[derive(Serialize)]
struct UserResponse {
    id: i32,
    username: String,
    email: String,
}

async fn login(
    State(pool): State<PgPool>,
    State(jwt_secret): State<String>,
    Json(req): Json<LoginRequest>,
) -> Result<Json<LoginResponse>, ApiError> {
    // Get user from database
    let user = sqlx::query!(
        "SELECT id, username, email, password_hash FROM users WHERE username = $1",
        req.username
    )
    .fetch_optional(&pool)
    .await?
    .ok_or_else(|| ApiError::Unauthorized("Invalid credentials".into()))?;

    // Verify password
    let is_valid = verify_password(&req.password, &user.password_hash)
        .map_err(|_| ApiError::InternalError("Password verification failed".into()))?;

    if !is_valid {
        return Err(ApiError::Unauthorized("Invalid credentials".into()));
    }

    // Create JWT
    let token = create_jwt(user.id, user.username.clone(), &jwt_secret)
        .map_err(|_| ApiError::InternalError("Failed to create token".into()))?;

    Ok(Json(LoginResponse {
        token,
        user: UserResponse {
            id: user.id,
            username: user.username,
            email: user.email,
        },
    }))
}
```

## Protected Routes with Middleware

### Create Auth Middleware

```rust
use axum::{
    extract::{Request, State},
    middleware::Next,
    response::Response,
    http::{StatusCode, header},
};

#[derive(Clone)]
pub struct AuthState {
    pub jwt_secret: String,
}

#[derive(Clone)]
pub struct CurrentUser {
    pub user_id: i32,
    pub username: String,
}

async fn auth_middleware(
    State(auth_state): State<AuthState>,
    mut req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    // Extract token from Authorization header
    let token = req
        .headers()
        .get(header::AUTHORIZATION)
        .and_then(|h| h.to_str().ok())
        .and_then(|h| h.strip_prefix("Bearer "))
        .ok_or(StatusCode::UNAUTHORIZED)?;

    // Verify token
    let claims = verify_jwt(token, &auth_state.jwt_secret)
        .map_err(|_| StatusCode::UNAUTHORIZED)?;

    // Add user to request extensions
    req.extensions_mut().insert(CurrentUser {
        user_id: claims.user_id,
        username: claims.username,
    });

    Ok(next.run(req).await)
}
```

### Using the Middleware

```rust
use axum::{
    Router,
    routing::get,
    middleware,
    extract::Extension,
};

async fn protected_route(
    Extension(user): Extension<CurrentUser>,
) -> String {
    format!("Hello, {}! Your user ID is {}", user.username, user.user_id)
}

fn create_router(auth_state: AuthState, pool: PgPool) -> Router {
    // Public routes
    let public_routes = Router::new()
        .route("/login", post(login))
        .route("/register", post(register));

    // Protected routes (require authentication)
    let protected_routes = Router::new()
        .route("/profile", get(get_profile))
        .route("/posts", post(create_post))
        .layer(middleware::from_fn_with_state(auth_state.clone(), auth_middleware));

    Router::new()
        .merge(public_routes)
        .merge(protected_routes)
        .with_state((auth_state, pool))
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let jwt_secret = std::env::var("JWT_SECRET")
        .unwrap_or_else(|_| "your-secret-key".to_string());

    let auth_state = AuthState { jwt_secret };

    let pool = /* create pool */;

    let app = create_router(auth_state, pool);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;
    axum::serve(listener, app).await?;

    Ok(())
}
```

## Extracting Current User

Create a custom extractor:

```rust
use axum::{
    async_trait,
    extract::FromRequestParts,
    http::{request::Parts, StatusCode},
};

#[async_trait]
impl<S> FromRequestParts<S> for CurrentUser
where
    S: Send + Sync,
{
    type Rejection = StatusCode;

    async fn from_request_parts(parts: &mut Parts, _state: &S) -> Result<Self, Self::Rejection> {
        parts
            .extensions
            .get::<CurrentUser>()
            .cloned()
            .ok_or(StatusCode::UNAUTHORIZED)
    }
}

// Now use it in handlers:
async fn get_profile(
    State(pool): State<PgPool>,
    user: CurrentUser,  // Automatically extracted!
) -> Result<Json<UserProfile>, ApiError> {
    let profile = sqlx::query_as!(
        UserProfile,
        "SELECT id, username, email, bio FROM users WHERE id = $1",
        user.user_id
    )
    .fetch_one(&pool)
    .await?;

    Ok(Json(profile))
}
```

## Refresh Tokens

Long-lived tokens for getting new access tokens:

```rust
use uuid::Uuid;

#[derive(Serialize)]
struct TokenPair {
    access_token: String,
    refresh_token: String,
}

async fn create_token_pair(
    user_id: i32,
    username: String,
    jwt_secret: &str,
    pool: &PgPool,
) -> Result<TokenPair, sqlx::Error> {
    // Short-lived access token (1 hour)
    let access_token = create_jwt_with_expiry(user_id, username.clone(), jwt_secret, 1)?;

    // Long-lived refresh token (30 days)
    let refresh_token = Uuid::new_v4().to_string();
    let expires_at = Utc::now() + Duration::days(30);

    // Store refresh token in database
    sqlx::query!(
        r#"
        INSERT INTO refresh_tokens (user_id, token, expires_at)
        VALUES ($1, $2, $3)
        "#,
        user_id,
        refresh_token,
        expires_at.naive_utc()
    )
    .execute(pool)
    .await?;

    Ok(TokenPair {
        access_token,
        refresh_token,
    })
}

async fn refresh_access_token(
    State(pool): State<PgPool>,
    State(jwt_secret): State<String>,
    Json(req): Json<RefreshRequest>,
) -> Result<Json<TokenResponse>, ApiError> {
    // Verify refresh token exists and is valid
    let token_record = sqlx::query!(
        r#"
        SELECT user_id, expires_at FROM refresh_tokens
        WHERE token = $1 AND expires_at > NOW()
        "#,
        req.refresh_token
    )
    .fetch_optional(&pool)
    .await?
    .ok_or_else(|| ApiError::Unauthorized("Invalid refresh token".into()))?;

    // Get user
    let user = sqlx::query!(
        "SELECT username FROM users WHERE id = $1",
        token_record.user_id
    )
    .fetch_one(&pool)
    .await?;

    // Create new access token
    let access_token = create_jwt(token_record.user_id, user.username, &jwt_secret)
        .map_err(|_| ApiError::InternalError("Failed to create token".into()))?;

    Ok(Json(TokenResponse { access_token }))
}
```

## Environment Variables for Secrets

**.env:**
```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/blog_dev
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
```

**Load in code:**
```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    dotenvy::dotenv().ok();

    let jwt_secret = std::env::var("JWT_SECRET")
        .expect("JWT_SECRET must be set");

    // Use jwt_secret...
}
```

## Security Best Practices

### 1. Keep Secrets Secret

❌ **Never:**
```rust
let secret = "hardcoded-secret";  // DON'T!
```

✅ **Always:**
```rust
let secret = std::env::var("JWT_SECRET").expect("JWT_SECRET not set");
```

### 2. Use Strong Secrets

```bash
# Generate a strong secret
openssl rand -base64 32
```

### 3. Short Expiration Times

```rust
// Access token: 1 hour
let expiration = now + Duration::hours(1);

// Refresh token: 30 days
let expiration = now + Duration::days(30);
```

### 4. HTTPS Only in Production

Never send tokens over HTTP in production!

### 5. Token Revocation

Store token IDs in database for revocation:

```rust
#[derive(Serialize, Deserialize)]
struct Claims {
    sub: String,
    exp: usize,
    iat: usize,
    jti: String,  // JWT ID for revocation
    user_id: i32,
}

async fn verify_token_not_revoked(jti: &str, pool: &PgPool) -> Result<bool, sqlx::Error> {
    let exists = sqlx::query!(
        "SELECT 1 FROM revoked_tokens WHERE jti = $1",
        jti
    )
    .fetch_optional(pool)
    .await?
    .is_some();

    Ok(!exists)
}
```

## Complete Example

```rust
use axum::{
    Router,
    routing::{get, post},
    middleware,
    extract::{State, Extension},
    http::StatusCode,
    Json,
};
use sqlx::PgPool;
use serde::{Deserialize, Serialize};

mod auth;  // Your auth module with JWT functions

#[derive(Clone)]
struct AppState {
    pool: PgPool,
    jwt_secret: String,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    dotenvy::dotenv().ok();

    let database_url = std::env::var("DATABASE_URL")?;
    let jwt_secret = std::env::var("JWT_SECRET")?;

    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await?;

    let state = AppState { pool, jwt_secret };

    let app = Router::new()
        // Public routes
        .route("/register", post(register))
        .route("/login", post(login))
        // Protected routes
        .route("/profile", get(get_profile))
        .route("/posts", post(create_post))
        .layer(middleware::from_fn_with_state(state.clone(), auth_middleware))
        .with_state(state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;
    println!("🚀 Server with JWT auth running!");

    axum::serve(listener, app).await?;

    Ok(())
}
```

## Testing with curl

```bash
# Register
curl -X POST http://localhost:3000/register \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","email":"alice@example.com","password":"SecurePass123"}'

# Login (get token)
TOKEN=$(curl -X POST http://localhost:3000/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"SecurePass123"}' \
  | jq -r '.token')

# Use token for protected route
curl http://localhost:3000/profile \
  -H "Authorization: Bearer $TOKEN"
```

## Key Takeaways

- ✅ JWTs enable stateless authentication
- ✅ Tokens contain signed user data (claims)
- ✅ Server verifies signature without database lookup
- ✅ Use short expiration times (1 hour for access tokens)
- ✅ Refresh tokens for long-lived sessions
- ✅ Store JWT secret in environment variables
- ✅ Use middleware to protect routes
- ✅ Extract current user in handlers automatically
- ✅ Consider token revocation for critical applications

**Next**: Authorization and role-based access control!

---

**Progress**: Module 14, Lesson 2 complete (69/90+ lessons total)
