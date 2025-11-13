# Module 14: Authentication & Security

# Lesson 3: Authorization & Role-Based Access Control

## Authentication vs Authorization

- **Authentication** = Who are you? (Login)
- **Authorization** = What can you do? (Permissions)

## Role-Based Access Control (RBAC)

Users have **roles**, roles have **permissions**.

**Example:**
- **Admin**: Can do everything
- **Editor**: Can create/edit/delete posts
- **User**: Can create/edit own posts, comment
- **Guest**: Can only view published posts

## Database Schema

```sql
-- Roles table
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- User roles junction table
CREATE TABLE user_roles (
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    role_id INTEGER REFERENCES roles(id) ON DELETE CASCADE,
    PRIMARY KEY (user_id, role_id)
);

-- Insert default roles
INSERT INTO roles (name) VALUES ('admin'), ('editor'), ('user');

-- Give alice admin role
INSERT INTO user_roles (user_id, role_id)
SELECT 1, id FROM roles WHERE name = 'admin';
```

## Adding Roles to JWT Claims

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct Claims {
    sub: String,
    exp: usize,
    iat: usize,
    user_id: i32,
    username: String,
    roles: Vec<String>,  // Add roles!
}

async fn create_jwt_with_roles(
    user_id: i32,
    username: String,
    pool: &PgPool,
    jwt_secret: &str,
) -> Result<String, Box<dyn std::error::Error>> {
    // Get user's roles
    let roles = sqlx::query!(
        r#"
        SELECT r.name
        FROM roles r
        JOIN user_roles ur ON r.id = ur.role_id
        WHERE ur.user_id = $1
        "#,
        user_id
    )
    .fetch_all(pool)
    .await?
    .into_iter()
    .map(|r| r.name)
    .collect();

    let now = Utc::now();
    let expiration = now + Duration::hours(1);

    let claims = Claims {
        sub: user_id.to_string(),
        exp: expiration.timestamp() as usize,
        iat: now.timestamp() as usize,
        user_id,
        username,
        roles,
    };

    let token = encode(
        &Header::default(),
        &claims,
        &EncodingKey::from_secret(jwt_secret.as_bytes())
    )?;

    Ok(token)
}
```

## CurrentUser with Roles

```rust
#[derive(Clone, Debug)]
pub struct CurrentUser {
    pub user_id: i32,
    pub username: String,
    pub roles: Vec<String>,
}

impl CurrentUser {
    pub fn has_role(&self, role: &str) -> bool {
        self.roles.iter().any(|r| r == role)
    }

    pub fn is_admin(&self) -> bool {
        self.has_role("admin")
    }

    pub fn is_editor(&self) -> bool {
        self.has_role("editor") || self.is_admin()
    }
}
```

## Role-Based Middleware

### Require Specific Role

```rust
use axum::{
    extract::Request,
    middleware::Next,
    response::Response,
    http::StatusCode,
};

async fn require_admin(
    Extension(user): Extension<CurrentUser>,
    req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    if user.is_admin() {
        Ok(next.run(req).await)
    } else {
        Err(StatusCode::FORBIDDEN)
    }
}

async fn require_editor(
    Extension(user): Extension<CurrentUser>,
    req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    if user.is_editor() {
        Ok(next.run(req).await)
    } else {
        Err(StatusCode::FORBIDDEN)
    }
}
```

### Using Role Middleware

```rust
use axum::{
    Router,
    routing::{get, post, delete},
    middleware,
};

fn create_router(state: AppState) -> Router {
    // Public routes
    let public = Router::new()
        .route("/login", post(login))
        .route("/posts", get(list_posts));

    // User routes (any authenticated user)
    let user_routes = Router::new()
        .route("/profile", get(get_profile))
        .layer(middleware::from_fn_with_state(state.clone(), auth_middleware));

    // Editor routes
    let editor_routes = Router::new()
        .route("/posts", post(create_post))
        .route("/posts/{id}", delete(delete_post))
        .layer(middleware::from_fn(require_editor))
        .layer(middleware::from_fn_with_state(state.clone(), auth_middleware));

    // Admin routes
    let admin_routes = Router::new()
        .route("/users", get(list_all_users))
        .route("/users/{id}/roles", post(assign_role))
        .layer(middleware::from_fn(require_admin))
        .layer(middleware::from_fn_with_state(state.clone(), auth_middleware));

    Router::new()
        .merge(public)
        .merge(user_routes)
        .nest("/admin", admin_routes)
        .nest("/editor", editor_routes)
        .with_state(state)
}
```

## Resource-Based Authorization

Check if user owns a resource:

```rust
async fn update_post(
    State(pool): State<PgPool>,
    user: CurrentUser,
    Path(post_id): Path<i32>,
    Json(update): Json<UpdatePost>,
) -> Result<Json<Post>, ApiError> {
    // Get post
    let post = sqlx::query!("SELECT user_id FROM posts WHERE id = $1", post_id)
        .fetch_optional(&pool)
        .await?
        .ok_or_else(|| ApiError::NotFound("Post not found".into()))?;

    // Check authorization
    if post.user_id != user.user_id && !user.is_admin() {
        return Err(ApiError::Forbidden("You don't own this post".into()));
    }

    // Proceed with update...
    let updated_post = sqlx::query_as!(
        Post,
        "UPDATE posts SET title = COALESCE($1, title), content = COALESCE($2, content) WHERE id = $3 RETURNING *",
        update.title,
        update.content,
        post_id
    )
    .fetch_one(&pool)
    .await?;

    Ok(Json(updated_post))
}
```

## Permission-Based System

More granular than roles:

```sql
-- Permissions table
CREATE TABLE permissions (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT
);

-- Role permissions junction
CREATE TABLE role_permissions (
    role_id INTEGER REFERENCES roles(id) ON DELETE CASCADE,
    permission_id INTEGER REFERENCES permissions(id) ON DELETE CASCADE,
    PRIMARY KEY (role_id, permission_id)
);

-- Insert permissions
INSERT INTO permissions (name, description) VALUES
    ('posts.create', 'Create new posts'),
    ('posts.edit.own', 'Edit own posts'),
    ('posts.edit.any', 'Edit any posts'),
    ('posts.delete.own', 'Delete own posts'),
    ('posts.delete.any', 'Delete any posts'),
    ('users.manage', 'Manage users'),
    ('roles.assign', 'Assign roles to users');

-- Assign permissions to admin role
INSERT INTO role_permissions (role_id, permission_id)
SELECT r.id, p.id
FROM roles r, permissions p
WHERE r.name = 'admin';

-- Assign some permissions to editor role
INSERT INTO role_permissions (role_id, permission_id)
SELECT r.id, p.id
FROM roles r, permissions p
WHERE r.name = 'editor'
  AND p.name IN ('posts.create', 'posts.edit.any', 'posts.delete.own');
```

### Load Permissions in JWT

```rust
#[derive(Debug, Serialize, Deserialize)]
struct Claims {
    sub: String,
    exp: usize,
    iat: usize,
    user_id: i32,
    username: String,
    permissions: Vec<String>,
}

async fn get_user_permissions(user_id: i32, pool: &PgPool) -> Result<Vec<String>, sqlx::Error> {
    let permissions = sqlx::query!(
        r#"
        SELECT DISTINCT p.name
        FROM permissions p
        JOIN role_permissions rp ON p.id = rp.permission_id
        JOIN user_roles ur ON rp.role_id = ur.role_id
        WHERE ur.user_id = $1
        "#,
        user_id
    )
    .fetch_all(pool)
    .await?
    .into_iter()
    .map(|r| r.name)
    .collect();

    Ok(permissions)
}
```

### Check Permissions

```rust
impl CurrentUser {
    pub fn can(&self, permission: &str) -> bool {
        self.permissions.iter().any(|p| p == permission)
    }
}

async fn delete_post(
    State(pool): State<PgPool>,
    user: CurrentUser,
    Path(post_id): Path<i32>,
) -> Result<StatusCode, ApiError> {
    let post = sqlx::query!("SELECT user_id FROM posts WHERE id = $1", post_id)
        .fetch_one(&pool)
        .await?;

    // Check permissions
    let can_delete = if post.user_id == user.user_id {
        user.can("posts.delete.own")
    } else {
        user.can("posts.delete.any")
    };

    if !can_delete {
        return Err(ApiError::Forbidden("Insufficient permissions".into()));
    }

    sqlx::query!("DELETE FROM posts WHERE id = $1", post_id)
        .execute(&pool)
        .await?;

    Ok(StatusCode::NO_CONTENT)
}
```

## Role Management Endpoints

### Assign Role to User

```rust
#[derive(Deserialize)]
struct AssignRoleRequest {
    role_name: String,
}

async fn assign_role(
    State(pool): State<PgPool>,
    user: CurrentUser,
    Path(target_user_id): Path<i32>,
    Json(req): Json<AssignRoleRequest>,
) -> Result<StatusCode, ApiError> {
    // Only admins can assign roles
    if !user.is_admin() {
        return Err(ApiError::Forbidden("Admin access required".into()));
    }

    // Get role ID
    let role = sqlx::query!("SELECT id FROM roles WHERE name = $1", req.role_name)
        .fetch_optional(&pool)
        .await?
        .ok_or_else(|| ApiError::NotFound("Role not found".into()))?;

    // Assign role
    sqlx::query!(
        "INSERT INTO user_roles (user_id, role_id) VALUES ($1, $2) ON CONFLICT DO NOTHING",
        target_user_id,
        role.id
    )
    .execute(&pool)
    .await?;

    Ok(StatusCode::OK)
}

async fn remove_role(
    State(pool): State<PgPool>,
    user: CurrentUser,
    Path((target_user_id, role_name)): Path<(i32, String)>,
) -> Result<StatusCode, ApiError> {
    if !user.is_admin() {
        return Err(ApiError::Forbidden("Admin access required".into()));
    }

    sqlx::query!(
        r#"
        DELETE FROM user_roles
        WHERE user_id = $1
          AND role_id = (SELECT id FROM roles WHERE name = $2)
        "#,
        target_user_id,
        role_name
    )
    .execute(&pool)
    .await?;

    Ok(StatusCode::NO_CONTENT)
}
```

## Audit Logging

Track who does what:

```sql
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id INTEGER,
    details JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_audit_log_user_id ON audit_log(user_id);
CREATE INDEX idx_audit_log_created_at ON audit_log(created_at);
```

```rust
async fn log_action(
    pool: &PgPool,
    user_id: i32,
    action: &str,
    resource_type: Option<&str>,
    resource_id: Option<i32>,
    details: Option<serde_json::Value>,
) -> Result<(), sqlx::Error> {
    sqlx::query!(
        r#"
        INSERT INTO audit_log (user_id, action, resource_type, resource_id, details)
        VALUES ($1, $2, $3, $4, $5)
        "#,
        user_id,
        action,
        resource_type,
        resource_id,
        details
    )
    .execute(pool)
    .await?;

    Ok(())
}

async fn delete_post_with_audit(
    State(pool): State<PgPool>,
    user: CurrentUser,
    Path(post_id): Path<i32>,
) -> Result<StatusCode, ApiError> {
    // Check permissions and delete...

    // Log the action
    log_action(
        &pool,
        user.user_id,
        "delete_post",
        Some("post"),
        Some(post_id),
        None,
    )
    .await?;

    Ok(StatusCode::NO_CONTENT)
}
```

## Complete Example

```rust
mod auth;
mod models;

use axum::{
    Router,
    routing::{get, post, put, delete},
    middleware,
    extract::{State, Path, Extension},
    http::StatusCode,
    Json,
};

#[derive(Clone)]
struct AppState {
    pool: PgPool,
    jwt_secret: String,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    dotenvy::dotenv().ok();

    let state = AppState {
        pool: /* create pool */,
        jwt_secret: std::env::var("JWT_SECRET")?,
    };

    let app = Router::new()
        // Public
        .route("/login", post(login))
        .route("/register", post(register))
        // Authenticated users
        .route("/posts", get(list_posts))
        .route("/profile", get(get_profile))
        .layer(middleware::from_fn_with_state(state.clone(), auth_middleware))
        // Editor routes
        .nest("/editor", Router::new()
            .route("/posts", post(create_post))
            .route("/posts/{id}", put(update_post).delete(delete_post))
            .layer(middleware::from_fn(require_editor))
        )
        // Admin routes
        .nest("/admin", Router::new()
            .route("/users", get(list_all_users))
            .route("/users/{id}/roles", post(assign_role))
            .route("/users/{id}/roles/{role}", delete(remove_role))
            .layer(middleware::from_fn(require_admin))
        )
        .with_state(state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;
    println!("🚀 Server with RBAC running!");

    axum::serve(listener, app).await?;

    Ok(())
}
```

## Testing

```bash
# Login as admin
ADMIN_TOKEN=$(curl -X POST http://localhost:3000/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123"}' \
  | jq -r '.token')

# Assign editor role to user 5
curl -X POST http://localhost:3000/admin/users/5/roles \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"role_name":"editor"}'

# Try to access admin route as regular user (should fail with 403)
curl http://localhost:3000/admin/users \
  -H "Authorization: Bearer $USER_TOKEN"
```

## Key Takeaways

- ✅ **Authentication** = Who you are
- ✅ **Authorization** = What you can do
- ✅ RBAC = Roles contain permissions
- ✅ Include roles/permissions in JWT claims
- ✅ Check permissions in handlers
- ✅ Use middleware for route-level authorization
- ✅ Resource-based checks (e.g., owns post)
- ✅ Audit log for security and compliance
- ✅ Return 403 Forbidden for unauthorized actions

**Next**: Security best practices and common vulnerabilities!

---

**Progress**: Module 14, Lesson 3 complete (70/90+ lessons total)
