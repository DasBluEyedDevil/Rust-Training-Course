# Module 14: Authentication & Security

# Lesson 5: Practice Project — Complete Authentication System

## Project Overview

Build a production-ready authentication system with:
- User registration with email verification
- Login with JWT tokens
- Role-based access control (RBAC)
- Password reset functionality
- Refresh tokens
- Rate limiting
- Audit logging
- Complete blog API with authorization

This integrates everything from Module 14!

## Project Setup

```bash
cargo new secure_blog_api
cd secure_blog_api
```

### Dependencies

```toml
[package]
name = "secure_blog_api"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.8"
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "chrono", "uuid"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
chrono = { version = "0.4", features = ["serde"] }
dotenvy = "0.15"
argon2 = "0.5"
jsonwebtoken = "9"
uuid = { version = "1.0", features = ["v4", "serde"] }
tower-http = { version = "0.5", features = ["cors", "trace"] }
rand = "0.8"
```

### Database Schema

Create migration:

```bash
sqlx database create
sqlx migrate add complete_auth_system
```

Edit migration file:

```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    email_verified BOOLEAN DEFAULT FALSE,
    failed_login_attempts INTEGER DEFAULT 0,
    locked_until TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Roles
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- User roles junction
CREATE TABLE user_roles (
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    role_id INTEGER REFERENCES roles(id) ON DELETE CASCADE,
    PRIMARY KEY (user_id, role_id)
);

-- Posts
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    content TEXT NOT NULL,
    published BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Refresh tokens
CREATE TABLE refresh_tokens (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Email verification tokens
CREATE TABLE email_verification_tokens (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Password reset tokens
CREATE TABLE password_reset_tokens (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    used BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Audit log
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id INTEGER,
    ip_address VARCHAR(45),
    details JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_slug ON posts(slug);
CREATE INDEX idx_posts_published ON posts(published);
CREATE INDEX idx_refresh_tokens_token ON refresh_tokens(token);
CREATE INDEX idx_refresh_tokens_user_id ON refresh_tokens(user_id);
CREATE INDEX idx_audit_log_user_id ON audit_log(user_id);
CREATE INDEX idx_audit_log_created_at ON audit_log(created_at);

-- Insert default roles
INSERT INTO roles (name) VALUES ('admin'), ('editor'), ('user');

-- Insert test admin user (password: Admin123!)
INSERT INTO users (username, email, password_hash, email_verified)
VALUES (
    'admin',
    'admin@example.com',
    '$argon2id$v=19$m=19456,t=2,p=1$...',  -- Hash of 'Admin123!'
    true
);

INSERT INTO user_roles (user_id, role_id)
SELECT 1, id FROM roles WHERE name = 'admin';
```

Run migration:

```bash
sqlx migrate run
```

## Project Structure

```
src/
├── main.rs
├── config.rs
├── models/
│   ├── mod.rs
│   ├── user.rs
│   ├── post.rs
│   └── claims.rs
├── handlers/
│   ├── mod.rs
│   ├── auth.rs
│   ├── posts.rs
│   └── admin.rs
├── middleware/
│   ├── mod.rs
│   ├── auth.rs
│   └── rate_limit.rs
├── utils/
│   ├── mod.rs
│   ├── password.rs
│   ├── jwt.rs
│   ├── audit.rs
│   └── email.rs
└── errors.rs
```

## Key Components

### src/config.rs

```rust
use serde::Deserialize;

#[derive(Clone, Debug)]
pub struct Config {
    pub database_url: String,
    pub jwt_secret: String,
    pub jwt_expiration_hours: i64,
    pub refresh_token_days: i64,
    pub server_host: String,
    pub server_port: u16,
}

impl Config {
    pub fn from_env() -> Result<Self, Box<dyn std::error::Error>> {
        dotenvy::dotenv().ok();

        let jwt_secret = std::env::var("JWT_SECRET")?;
        if jwt_secret.len() < 32 {
            return Err("JWT_SECRET must be at least 32 characters".into());
        }

        Ok(Self {
            database_url: std::env::var("DATABASE_URL")?,
            jwt_secret,
            jwt_expiration_hours: std::env::var("JWT_EXPIRATION_HOURS")
                .unwrap_or_else(|_| "1".to_string())
                .parse()?,
            refresh_token_days: std::env::var("REFRESH_TOKEN_DAYS")
                .unwrap_or_else(|_| "30".to_string())
                .parse()?,
            server_host: std::env::var("SERVER_HOST")
                .unwrap_or_else(|_| "0.0.0.0".to_string()),
            server_port: std::env::var("SERVER_PORT")
                .unwrap_or_else(|_| "3000".to_string())
                .parse()?,
        })
    }
}
```

### src/utils/password.rs

```rust
use argon2::{
    password_hash::{rand_core::OsRng, PasswordHash, PasswordHasher, PasswordVerifier, SaltString},
    Argon2,
};

pub fn hash_password(password: &str) -> Result<String, argon2::password_hash::Error> {
    let salt = SaltString::generate(&mut OsRng);
    let argon2 = Argon2::default();
    let password_hash = argon2.hash_password(password.as_bytes(), &salt)?.to_string();
    Ok(password_hash)
}

pub fn verify_password(password: &str, hash: &str) -> Result<bool, argon2::password_hash::Error> {
    let parsed_hash = PasswordHash::new(hash)?;
    let argon2 = Argon2::default();
    match argon2.verify_password(password.as_bytes(), &parsed_hash) {
        Ok(()) => Ok(true),
        Err(argon2::password_hash::Error::Password) => Ok(false),
        Err(e) => Err(e),
    }
}

pub fn validate_password(password: &str) -> Result<(), String> {
    if password.len() < 8 {
        return Err("Password must be at least 8 characters".into());
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
    Ok(())
}
```

### src/utils/jwt.rs

```rust
use chrono::{Duration, Utc};
use jsonwebtoken::{decode, encode, DecodingKey, EncodingKey, Header, Validation};
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct Claims {
    pub sub: String,
    pub exp: usize,
    pub iat: usize,
    pub user_id: i32,
    pub username: String,
    pub roles: Vec<String>,
}

pub fn create_jwt(
    user_id: i32,
    username: String,
    roles: Vec<String>,
    secret: &str,
    expiration_hours: i64,
) -> Result<String, jsonwebtoken::errors::Error> {
    let now = Utc::now();
    let expiration = now + Duration::hours(expiration_hours);

    let claims = Claims {
        sub: user_id.to_string(),
        exp: expiration.timestamp() as usize,
        iat: now.timestamp() as usize,
        user_id,
        username,
        roles,
    };

    encode(&Header::default(), &claims, &EncodingKey::from_secret(secret.as_bytes()))
}

pub fn verify_jwt(token: &str, secret: &str) -> Result<Claims, jsonwebtoken::errors::Error> {
    let token_data = decode::<Claims>(
        token,
        &DecodingKey::from_secret(secret.as_bytes()),
        &Validation::default(),
    )?;

    Ok(token_data.claims)
}
```

### src/handlers/auth.rs (Complete)

```rust
use axum::{extract::State, http::StatusCode, Json};
use chrono::{Duration, Utc};
use serde::{Deserialize, Serialize};
use sqlx::PgPool;
use uuid::Uuid;

use crate::{
    config::Config,
    errors::ApiError,
    models::user::User,
    utils::{audit, jwt, password},
};

#[derive(Deserialize)]
pub struct RegisterRequest {
    pub username: String,
    pub email: String,
    pub password: String,
}

#[derive(Serialize)]
pub struct RegisterResponse {
    pub user: UserResponse,
    pub message: String,
}

#[derive(Serialize)]
pub struct UserResponse {
    pub id: i32,
    pub username: String,
    pub email: String,
    pub email_verified: bool,
}

pub async fn register(
    State(pool): State<PgPool>,
    Json(req): Json<RegisterRequest>,
) -> Result<(StatusCode, Json<RegisterResponse>), ApiError> {
    // Validate password
    password::validate_password(&req.password)
        .map_err(|e| ApiError::BadRequest(e))?;

    // Hash password
    let password_hash = password::hash_password(&req.password)
        .map_err(|_| ApiError::InternalError("Failed to hash password".into()))?;

    // Insert user
    let user = sqlx::query_as!(
        User,
        r#"
        INSERT INTO users (username, email, password_hash)
        VALUES ($1, $2, $3)
        RETURNING id, username, email, password_hash, email_verified,
                  failed_login_attempts, locked_until, created_at, updated_at
        "#,
        req.username,
        req.email,
        password_hash
    )
    .fetch_one(&pool)
    .await
    .map_err(|e| match e {
        sqlx::Error::Database(db_err) if db_err.is_unique_violation() => {
            ApiError::BadRequest("Username or email already exists".into())
        }
        _ => ApiError::from(e),
    })?;

    // Assign default 'user' role
    sqlx::query!(
        "INSERT INTO user_roles (user_id, role_id) SELECT $1, id FROM roles WHERE name = 'user'",
        user.id
    )
    .execute(&pool)
    .await?;

    // Create email verification token
    let verification_token = Uuid::new_v4().to_string();
    let expires_at = Utc::now() + Duration::hours(24);

    sqlx::query!(
        "INSERT INTO email_verification_tokens (user_id, token, expires_at) VALUES ($1, $2, $3)",
        user.id,
        verification_token,
        expires_at.naive_utc()
    )
    .execute(&pool)
    .await?;

    // In production: Send email with verification link
    // send_verification_email(&user.email, &verification_token).await?;

    Ok((
        StatusCode::CREATED,
        Json(RegisterResponse {
            user: UserResponse {
                id: user.id,
                username: user.username,
                email: user.email,
                email_verified: user.email_verified,
            },
            message: "Registration successful. Please verify your email.".to_string(),
        }),
    ))
}

#[derive(Deserialize)]
pub struct LoginRequest {
    pub username: String,
    pub password: String,
}

#[derive(Serialize)]
pub struct LoginResponse {
    pub access_token: String,
    pub refresh_token: String,
    pub user: UserResponse,
}

pub async fn login(
    State(pool): State<PgPool>,
    State(config): State<Config>,
    Json(req): Json<LoginRequest>,
) -> Result<Json<LoginResponse>, ApiError> {
    // Get user
    let user = sqlx::query_as!(
        User,
        "SELECT * FROM users WHERE username = $1",
        req.username
    )
    .fetch_optional(&pool)
    .await?
    .ok_or_else(|| ApiError::Unauthorized("Invalid credentials".into()))?;

    // Check if account is locked
    if let Some(locked_until) = user.locked_until {
        if locked_until.and_utc() > Utc::now() {
            return Err(ApiError::Forbidden("Account is locked".into()));
        }
    }

    // Verify password
    let is_valid = password::verify_password(&req.password, &user.password_hash)
        .map_err(|_| ApiError::InternalError("Password verification failed".into()))?;

    if !is_valid {
        // Increment failed attempts
        let new_attempts = user.failed_login_attempts + 1;
        let locked_until = if new_attempts >= 5 {
            Some(Utc::now() + Duration::minutes(15))
        } else {
            None
        };

        sqlx::query!(
            "UPDATE users SET failed_login_attempts = $1, locked_until = $2 WHERE id = $3",
            new_attempts,
            locked_until.map(|dt| dt.naive_utc()),
            user.id
        )
        .execute(&pool)
        .await?;

        return Err(ApiError::Unauthorized("Invalid credentials".into()));
    }

    // Reset failed attempts on successful login
    sqlx::query!(
        "UPDATE users SET failed_login_attempts = 0, locked_until = NULL WHERE id = $1",
        user.id
    )
    .execute(&pool)
    .await?;

    // Get user roles
    let roles = sqlx::query!(
        r#"
        SELECT r.name FROM roles r
        JOIN user_roles ur ON r.id = ur.role_id
        WHERE ur.user_id = $1
        "#,
        user.id
    )
    .fetch_all(&pool)
    .await?
    .into_iter()
    .map(|r| r.name)
    .collect();

    // Create JWT
    let access_token = jwt::create_jwt(
        user.id,
        user.username.clone(),
        roles,
        &config.jwt_secret,
        config.jwt_expiration_hours,
    )
    .map_err(|_| ApiError::InternalError("Failed to create token".into()))?;

    // Create refresh token
    let refresh_token = Uuid::new_v4().to_string();
    let expires_at = Utc::now() + Duration::days(config.refresh_token_days);

    sqlx::query!(
        "INSERT INTO refresh_tokens (user_id, token, expires_at) VALUES ($1, $2, $3)",
        user.id,
        refresh_token,
        expires_at.naive_utc()
    )
    .execute(&pool)
    .await?;

    Ok(Json(LoginResponse {
        access_token,
        refresh_token,
        user: UserResponse {
            id: user.id,
            username: user.username,
            email: user.email,
            email_verified: user.email_verified,
        },
    }))
}
```

### src/middleware/auth.rs

```rust
use axum::{
    extract::{Request, State},
    http::{header, StatusCode},
    middleware::Next,
    response::Response,
};

use crate::{config::Config, models::user::CurrentUser, utils::jwt};

pub async fn auth_middleware(
    State(config): State<Config>,
    mut req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    let token = req
        .headers()
        .get(header::AUTHORIZATION)
        .and_then(|h| h.to_str().ok())
        .and_then(|h| h.strip_prefix("Bearer "))
        .ok_or(StatusCode::UNAUTHORIZED)?;

    let claims = jwt::verify_jwt(token, &config.jwt_secret).map_err(|_| StatusCode::UNAUTHORIZED)?;

    req.extensions_mut().insert(CurrentUser {
        user_id: claims.user_id,
        username: claims.username,
        roles: claims.roles,
    });

    Ok(next.run(req).await)
}

pub async fn require_role(
    role: String,
    Extension(user): Extension<CurrentUser>,
    req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    if user.has_role(&role) {
        Ok(next.run(req).await)
    } else {
        Err(StatusCode::FORBIDDEN)
    }
}
```

## Testing the Complete System

```bash
# 1. Register
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "email": "test@example.com",
    "password": "SecurePass123"
  }'

# 2. Login
LOGIN_RESPONSE=$(curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "SecurePass123"
  }')

TOKEN=$(echo $LOGIN_RESPONSE | jq -r '.access_token')
REFRESH=$(echo $LOGIN_RESPONSE | jq -r '.refresh_token')

# 3. Access protected route
curl http://localhost:3000/posts \
  -H "Authorization: Bearer $TOKEN"

# 4. Create post (requires authentication)
curl -X POST http://localhost:3000/posts \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My First Post",
    "content": "This is the content"
  }'

# 5. Refresh access token
curl -X POST http://localhost:3000/auth/refresh \
  -H "Content-Type: application/json" \
  -d "{\"refresh_token\": \"$REFRESH\"}"

# 6. Admin endpoint (will fail for non-admin)
curl http://localhost:3000/admin/users \
  -H "Authorization: Bearer $TOKEN"
```

## Key Features Implemented

✅ User registration with validation
✅ Secure password hashing (Argon2)
✅ Email verification tokens
✅ Login with JWT
✅ Refresh tokens (30-day expiry)
✅ Role-based access control
✅ Account lockout after failed attempts
✅ Password reset functionality
✅ Audit logging
✅ Rate limiting
✅ CORS configuration
✅ Input validation
✅ SQL injection prevention
✅ Authorization checks

## Challenge Extensions

Add these features:

1. **Email service integration** - Actually send verification emails
2. **Two-factor authentication (2FA)** - TOTP codes
3. **OAuth integration** - Login with Google/GitHub
4. **Session management** - View active sessions, logout all devices
5. **API key authentication** - For programmatic access
6. **Webhook notifications** - Security events

## Key Takeaways

- ✅ Complete production-ready auth system
- ✅ Multiple layers of security
- ✅ Proper password handling
- ✅ JWT with refresh tokens
- ✅ Role-based authorization
- ✅ Account protection (lockout, rate limiting)
- ✅ Audit logging for security
- ✅ Environment-based configuration
- ✅ Database-backed everything (persistent)

**Module 14 Complete!** You now have a solid understanding of authentication and security in Rust web applications.

---

**Progress**: Module 14 complete! (72/90+ lessons total)
