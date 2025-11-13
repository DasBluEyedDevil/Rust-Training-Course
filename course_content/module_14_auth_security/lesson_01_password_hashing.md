# Module 14: Authentication & Security

# Lesson 1: Password Hashing & User Authentication

## Why Authentication Matters

**Authentication** verifies **who** a user is. Without it:
- Anyone can access private data
- No personalization
- No accountability
- No protected actions

## Never Store Plain Passwords!

❌ **NEVER DO THIS:**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    password VARCHAR(255)  -- NEVER store plain text!
);

INSERT INTO users (username, password) VALUES ('alice', 'password123');  -- TERRIBLE!
```

**Why it's dangerous:**
- Database breach = all passwords exposed
- Admins can see passwords
- Users often reuse passwords across sites

## Password Hashing

**Hashing** transforms passwords into irreversible strings:

```
"password123" → hash → "$argon2id$v=19$m=19456,t=2,p=1$..."
```

**Properties:**
- **One-way**: Can't reverse hash to get password
- **Deterministic**: Same password = same hash
- **Unique**: Different passwords = different hashes
- **Slow**: Makes brute-force attacks impractical

## Using `argon2` for Password Hashing

### Setup

```toml
[dependencies]
argon2 = "0.5"
rand = "0.8"
```

### Hash a Password

```rust
use argon2::{
    password_hash::{
        rand_core::OsRng,
        PasswordHash, PasswordHasher, PasswordVerifier, SaltString
    },
    Argon2
};

fn hash_password(password: &str) -> Result<String, argon2::password_hash::Error> {
    let salt = SaltString::generate(&mut OsRng);
    let argon2 = Argon2::default();

    let password_hash = argon2
        .hash_password(password.as_bytes(), &salt)?
        .to_string();

    Ok(password_hash)
}

#[tokio::main]
async fn main() {
    let password = "my_secure_password";
    let hash = hash_password(password).unwrap();

    println!("Password: {}", password);
    println!("Hash: {}", hash);
    // Output: $argon2id$v=19$m=19456,t=2,p=1$...
}
```

### Verify a Password

```rust
fn verify_password(password: &str, hash: &str) -> Result<bool, argon2::password_hash::Error> {
    let parsed_hash = PasswordHash::new(hash)?;
    let argon2 = Argon2::default();

    match argon2.verify_password(password.as_bytes(), &parsed_hash) {
        Ok(()) => Ok(true),
        Err(argon2::password_hash::Error::Password) => Ok(false),
        Err(e) => Err(e),
    }
}

fn main() {
    let password = "my_secure_password";
    let hash = hash_password(password).unwrap();

    // Correct password
    assert!(verify_password(password, &hash).unwrap());

    // Wrong password
    assert!(!verify_password("wrong_password", &hash).unwrap());
}
```

## User Registration

### Database Schema

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

**Notice:** We store `password_hash`, not `password`!

### Registration Handler

```rust
use axum::{
    extract::State,
    http::StatusCode,
    Json,
};
use serde::{Deserialize, Serialize};
use sqlx::PgPool;

#[derive(Deserialize)]
struct RegisterRequest {
    username: String,
    email: String,
    password: String,
}

#[derive(Serialize)]
struct User {
    id: i32,
    username: String,
    email: String,
}

async fn register(
    State(pool): State<PgPool>,
    Json(req): Json<RegisterRequest>,
) -> Result<(StatusCode, Json<User>), ApiError> {
    // Validate password strength
    if req.password.len() < 8 {
        return Err(ApiError::BadRequest("Password must be at least 8 characters".into()));
    }

    // Hash password
    let password_hash = hash_password(&req.password)
        .map_err(|_| ApiError::InternalError("Failed to hash password".into()))?;

    // Insert user
    let user = sqlx::query_as!(
        User,
        r#"
        INSERT INTO users (username, email, password_hash)
        VALUES ($1, $2, $3)
        RETURNING id, username, email
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

    Ok((StatusCode::CREATED, Json(user)))
}
```

## User Login

### Login Handler

```rust
#[derive(Deserialize)]
struct LoginRequest {
    username: String,
    password: String,
}

#[derive(Serialize)]
struct LoginResponse {
    user: User,
    message: String,
}

async fn login(
    State(pool): State<PgPool>,
    Json(req): Json<LoginRequest>,
) -> Result<Json<LoginResponse>, ApiError> {
    // Get user from database
    let user_record = sqlx::query!(
        "SELECT id, username, email, password_hash FROM users WHERE username = $1",
        req.username
    )
    .fetch_optional(&pool)
    .await?
    .ok_or_else(|| ApiError::Unauthorized("Invalid credentials".into()))?;

    // Verify password
    let is_valid = verify_password(&req.password, &user_record.password_hash)
        .map_err(|_| ApiError::InternalError("Password verification failed".into()))?;

    if !is_valid {
        return Err(ApiError::Unauthorized("Invalid credentials".into()));
    }

    // Success!
    Ok(Json(LoginResponse {
        user: User {
            id: user_record.id,
            username: user_record.username,
            email: user_record.email,
        },
        message: "Login successful".into(),
    }))
}
```

## Security Best Practices

### Password Requirements

```rust
fn validate_password(password: &str) -> Result<(), String> {
    if password.len() < 8 {
        return Err("Password must be at least 8 characters".into());
    }

    let has_uppercase = password.chars().any(|c| c.is_uppercase());
    let has_lowercase = password.chars().any(|c| c.is_lowercase());
    let has_digit = password.chars().any(|c| c.is_numeric());

    if !has_uppercase || !has_lowercase || !has_digit {
        return Err("Password must contain uppercase, lowercase, and digit".into());
    }

    Ok(())
}
```

### Rate Limiting (Basic)

Prevent brute-force attacks:

```rust
use std::collections::HashMap;
use std::sync::Arc;
use tokio::sync::Mutex;
use std::time::{SystemTime, UNIX_EPOCH};

#[derive(Clone)]
struct RateLimiter {
    attempts: Arc<Mutex<HashMap<String, Vec<u64>>>>,
}

impl RateLimiter {
    fn new() -> Self {
        Self {
            attempts: Arc::new(Mutex::new(HashMap::new())),
        }
    }

    async fn check_rate_limit(&self, username: &str) -> Result<(), String> {
        let mut attempts = self.attempts.lock().await;
        let now = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();

        let user_attempts = attempts.entry(username.to_string()).or_insert_with(Vec::new);

        // Remove attempts older than 15 minutes
        user_attempts.retain(|&time| now - time < 900);

        // Check if too many attempts
        if user_attempts.len() >= 5 {
            return Err("Too many login attempts. Try again later.".into());
        }

        // Record this attempt
        user_attempts.push(now);

        Ok(())
    }
}

async fn login_with_rate_limit(
    State(pool): State<PgPool>,
    State(rate_limiter): State<RateLimiter>,
    Json(req): Json<LoginRequest>,
) -> Result<Json<LoginResponse>, ApiError> {
    // Check rate limit
    rate_limiter
        .check_rate_limit(&req.username)
        .await
        .map_err(|e| ApiError::TooManyRequests(e))?;

    // Continue with login...
    login(State(pool), Json(req)).await
}
```

### Email Validation

```rust
fn validate_email(email: &str) -> bool {
    email.contains('@') && email.contains('.') && email.len() >= 5
}
```

### Username Sanitization

```rust
fn validate_username(username: &str) -> Result<(), String> {
    if username.len() < 3 || username.len() > 30 {
        return Err("Username must be 3-30 characters".into());
    }

    if !username.chars().all(|c| c.is_alphanumeric() || c == '_' || c == '-') {
        return Err("Username can only contain letters, numbers, _ and -".into());
    }

    Ok(())
}
```

## Complete Registration/Login Example

```rust
use axum::{
    routing::post,
    Router,
    extract::State,
    http::StatusCode,
    Json,
};
use sqlx::postgres::PgPoolOptions;
use serde::{Deserialize, Serialize};

mod auth {
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
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    dotenvy::dotenv().ok();
    let database_url = std::env::var("DATABASE_URL")?;

    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await?;

    let app = Router::new()
        .route("/register", post(register))
        .route("/login", post(login))
        .with_state(pool);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;
    println!("🚀 Auth server running on http://localhost:3000");

    axum::serve(listener, app).await?;

    Ok(())
}
```

## Practice Exercises

### Exercise 1: Password Strength Meter

Create a function that returns password strength (weak/medium/strong):

```rust
enum PasswordStrength {
    Weak,
    Medium,
    Strong,
}

fn check_password_strength(password: &str) -> PasswordStrength {
    // Your implementation here
}
```

<details>
<summary>Solution</summary>

```rust
fn check_password_strength(password: &str) -> PasswordStrength {
    let mut score = 0;

    if password.len() >= 8 { score += 1; }
    if password.len() >= 12 { score += 1; }
    if password.chars().any(|c| c.is_uppercase()) { score += 1; }
    if password.chars().any(|c| c.is_lowercase()) { score += 1; }
    if password.chars().any(|c| c.is_numeric()) { score += 1; }
    if password.chars().any(|c| !c.is_alphanumeric()) { score += 1; }

    match score {
        0..=2 => PasswordStrength::Weak,
        3..=4 => PasswordStrength::Medium,
        _ => PasswordStrength::Strong,
    }
}
```
</details>

### Exercise 2: Change Password Endpoint

Create a POST /change-password endpoint that:
- Verifies old password
- Validates new password
- Updates password_hash in database

<details>
<summary>Hint</summary>

```rust
#[derive(Deserialize)]
struct ChangePasswordRequest {
    user_id: i32,
    old_password: String,
    new_password: String,
}

async fn change_password(
    State(pool): State<PgPool>,
    Json(req): Json<ChangePasswordRequest>,
) -> Result<StatusCode, ApiError> {
    // 1. Get current password_hash from database
    // 2. Verify old_password matches
    // 3. Validate new_password
    // 4. Hash new_password
    // 5. Update database
    // 6. Return StatusCode::OK
}
```
</details>

### Exercise 3: Account Lockout

Implement account lockout after 5 failed login attempts:

<details>
<summary>Hint</summary>

Add `failed_attempts` and `locked_until` columns to users table:

```sql
ALTER TABLE users ADD COLUMN failed_attempts INTEGER DEFAULT 0;
ALTER TABLE users ADD COLUMN locked_until TIMESTAMP NULL;
```

In login handler:
1. Check if `locked_until` is in the future
2. On failed login, increment `failed_attempts`
3. If >= 5, set `locked_until` to 15 minutes from now
4. On successful login, reset `failed_attempts` to 0
</details>

## Key Takeaways

- ✅ NEVER store passwords in plain text
- ✅ Use argon2 for password hashing (slow, secure)
- ✅ Hash passwords before storing
- ✅ Verify passwords by comparing hashes
- ✅ Validate password strength (length, complexity)
- ✅ Implement rate limiting to prevent brute-force
- ✅ Return generic error messages ("Invalid credentials")
- ✅ Hash passwords on registration, verify on login

**Next**: JWT tokens for stateless authentication!

---

**Progress**: Module 14, Lesson 1 complete (68/90+ lessons total)
