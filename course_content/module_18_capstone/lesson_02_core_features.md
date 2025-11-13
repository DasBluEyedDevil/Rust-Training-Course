# Module 18: Full-Stack Capstone Project

# Lesson 2: Implementing Core Features

This lesson provides a complete implementation guide. Refer to previous modules for detailed explanations.

## Posts Handler

```rust
// src/handlers/posts.rs
use axum::{extract::{State, Path}, Json};
use sqlx::PgPool;

pub async fn create_post(
    State(pool): State<PgPool>,
    user: CurrentUser,
    Json(req): Json<CreatePostRequest>,
) -> Result<Json<Post>, ApiError> {
    let post = sqlx::query_as!(
        Post,
        r#"
        INSERT INTO posts (user_id, content, image_url)
        VALUES ($1, $2, $3)
        RETURNING *
        "#,
        user.user_id,
        req.content,
        req.image_url
    )
    .fetch_one(&pool)
    .await?;

    Ok(Json(post))
}

pub async fn like_post(
    State(pool): State<PgPool>,
    user: CurrentUser,
    Path(post_id): Path<i32>,
) -> Result<StatusCode, ApiError> {
    // Insert like
    sqlx::query!(
        "INSERT INTO post_likes (user_id, post_id) VALUES ($1, $2) ON CONFLICT DO NOTHING",
        user.user_id,
        post_id
    )
    .execute(&pool)
    .await?;

    // Increment count
    sqlx::query!("UPDATE posts SET likes_count = likes_count + 1 WHERE id = $1", post_id)
        .execute(&pool)
        .await?;

    Ok(StatusCode::OK)
}
```

## Follow System

```rust
pub async fn follow_user(
    State(pool): State<PgPool>,
    user: CurrentUser,
    Path(username): Path<String>,
) -> Result<StatusCode, ApiError> {
    let target_user = get_user_by_username(&pool, &username).await?;

    sqlx::query!(
        "INSERT INTO follows (follower_id, following_id) VALUES ($1, $2)",
        user.user_id,
        target_user.id
    )
    .execute(&pool)
    .await?;

    Ok(StatusCode::OK)
}
```

## Feed Algorithm

```rust
pub async fn get_feed(
    State(pool): State<PgPool>,
    user: CurrentUser,
) -> Result<Json<Vec<PostWithAuthor>>, ApiError> {
    let posts = sqlx::query_as!(
        PostWithAuthor,
        r#"
        SELECT p.*, u.username, u.avatar_url
        FROM posts p
        JOIN users u ON p.user_id = u.id
        WHERE p.user_id IN (
            SELECT following_id FROM follows WHERE follower_id = $1
        ) OR p.user_id = $1
        ORDER BY p.created_at DESC
        LIMIT 50
        "#,
        user.user_id
    )
    .fetch_all(&pool)
    .await?;

    Ok(Json(posts))
}
```

**Key Takeaways:**
- ✅ Implement CRUD for posts
- ✅ Like/unlike functionality
- ✅ Follow system
- ✅ Personalized feed

**Next**: Real-time features and deployment!

---

**Progress**: Module 18, Lesson 2 complete (83/90+ lessons total)
