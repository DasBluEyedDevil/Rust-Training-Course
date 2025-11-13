# Module 13: Database Integration

# Lesson 4: Advanced Database Patterns

## Joins - Combining Related Data

### Inner Join

Get posts with their author information:

```rust
#[derive(Debug, Serialize)]
struct PostWithAuthor {
    id: i32,
    title: String,
    content: String,
    published: bool,
    author_id: i32,
    author_name: String,
    author_email: String,
}

async fn get_posts_with_authors(
    pool: &PgPool,
) -> Result<Vec<PostWithAuthor>, sqlx::Error> {
    let posts = sqlx::query_as!(
        PostWithAuthor,
        r#"
        SELECT
            p.id,
            p.title,
            p.content,
            p.published,
            u.id as author_id,
            u.username as author_name,
            u.email as author_email
        FROM posts p
        INNER JOIN users u ON p.user_id = u.id
        ORDER BY p.created_at DESC
        "#
    )
    .fetch_all(pool)
    .await?;

    Ok(posts)
}
```

### Left Join

Get all users with their post counts (including users with zero posts):

```rust
#[derive(Debug, Serialize)]
struct UserWithPostCount {
    id: i32,
    username: String,
    email: String,
    post_count: Option<i64>,
}

async fn get_users_with_post_counts(
    pool: &PgPool,
) -> Result<Vec<UserWithPostCount>, sqlx::Error> {
    let users = sqlx::query_as!(
        UserWithPostCount,
        r#"
        SELECT
            u.id,
            u.username,
            u.email,
            COUNT(p.id) as "post_count: i64"
        FROM users u
        LEFT JOIN posts p ON u.id = p.user_id
        GROUP BY u.id, u.username, u.email
        ORDER BY post_count DESC
        "#
    )
    .fetch_all(pool)
    .await?;

    Ok(users)
}
```

## Aggregations

### COUNT, SUM, AVG, MIN, MAX

```rust
#[derive(Debug, Serialize)]
struct PostStats {
    total_posts: i64,
    published_posts: i64,
    draft_posts: i64,
    total_users: i64,
}

async fn get_stats(pool: &PgPool) -> Result<PostStats, sqlx::Error> {
    let stats = sqlx::query!(
        r#"
        SELECT
            COUNT(p.id) as "total_posts!",
            COUNT(p.id) FILTER (WHERE p.published = true) as "published_posts!",
            COUNT(p.id) FILTER (WHERE p.published = false) as "draft_posts!",
            COUNT(DISTINCT p.user_id) as "total_users!"
        FROM posts p
        "#
    )
    .fetch_one(pool)
    .await?;

    Ok(PostStats {
        total_posts: stats.total_posts,
        published_posts: stats.published_posts,
        draft_posts: stats.draft_posts,
        total_users: stats.total_users,
    })
}
```

### GROUP BY

Posts per user:

```rust
#[derive(Debug, Serialize)]
struct UserPostStats {
    user_id: i32,
    username: String,
    post_count: i64,
    published_count: i64,
}

async fn posts_per_user(pool: &PgPool) -> Result<Vec<UserPostStats>, sqlx::Error> {
    let stats = sqlx::query_as!(
        UserPostStats,
        r#"
        SELECT
            u.id as user_id,
            u.username,
            COUNT(p.id) as "post_count!",
            COUNT(p.id) FILTER (WHERE p.published = true) as "published_count!"
        FROM users u
        LEFT JOIN posts p ON u.id = p.user_id
        GROUP BY u.id, u.username
        ORDER BY post_count DESC
        "#
    )
    .fetch_all(pool)
    .await?;

    Ok(stats)
}
```

## Subqueries

Get users who have published posts:

```rust
async fn users_with_published_posts(pool: &PgPool) -> Result<Vec<User>, sqlx::Error> {
    let users = sqlx::query_as!(
        User,
        r#"
        SELECT id, username, email, created_at
        FROM users
        WHERE id IN (
            SELECT DISTINCT user_id
            FROM posts
            WHERE published = true
        )
        ORDER BY username
        "#
    )
    .fetch_all(pool)
    .await?;

    Ok(users)
}
```

## Full-Text Search

PostgreSQL has powerful full-text search capabilities:

### Setup

```sql
-- Add migration
ALTER TABLE posts ADD COLUMN search_vector tsvector;

CREATE INDEX idx_posts_search ON posts USING GIN(search_vector);

-- Update function to maintain search vector
CREATE FUNCTION posts_search_trigger() RETURNS trigger AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('english', COALESCE(NEW.title, '')), 'A') ||
        setweight(to_tsvector('english', COALESCE(NEW.content, '')), 'B');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER posts_search_update
    BEFORE INSERT OR UPDATE ON posts
    FOR EACH ROW EXECUTE FUNCTION posts_search_trigger();

-- Update existing rows
UPDATE posts SET search_vector =
    setweight(to_tsvector('english', COALESCE(title, '')), 'A') ||
    setweight(to_tsvector('english', COALESCE(content, '')), 'B');
```

### Search Query

```rust
async fn search_posts(
    pool: &PgPool,
    query: &str,
) -> Result<Vec<Post>, sqlx::Error> {
    let posts = sqlx::query_as!(
        Post,
        r#"
        SELECT *
        FROM posts
        WHERE search_vector @@ plainto_tsquery('english', $1)
        ORDER BY ts_rank(search_vector, plainto_tsquery('english', $1)) DESC
        "#,
        query
    )
    .fetch_all(pool)
    .await?;

    Ok(posts)
}
```

## Many-to-Many Relationships

### Example: Posts and Tags

Migration:

```sql
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE post_tags (
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    tag_id INTEGER REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);

CREATE INDEX idx_post_tags_post_id ON post_tags(post_id);
CREATE INDEX idx_post_tags_tag_id ON post_tags(tag_id);
```

### Get Post with Tags

```rust
#[derive(Debug, Serialize)]
struct PostWithTags {
    #[serde(flatten)]
    post: Post,
    tags: Vec<String>,
}

async fn get_post_with_tags(
    pool: &PgPool,
    post_id: i32,
) -> Result<Option<PostWithTags>, sqlx::Error> {
    // Get post
    let post = sqlx::query_as!(
        Post,
        "SELECT * FROM posts WHERE id = $1",
        post_id
    )
    .fetch_optional(pool)
    .await?;

    if let Some(post) = post {
        // Get tags for post
        let tags = sqlx::query!(
            r#"
            SELECT t.name
            FROM tags t
            INNER JOIN post_tags pt ON t.id = pt.tag_id
            WHERE pt.post_id = $1
            "#,
            post_id
        )
        .fetch_all(pool)
        .await?
        .into_iter()
        .map(|r| r.name)
        .collect();

        Ok(Some(PostWithTags { post, tags }))
    } else {
        Ok(None)
    }
}
```

### Add Tag to Post

```rust
async fn add_tag_to_post(
    pool: &PgPool,
    post_id: i32,
    tag_name: &str,
) -> Result<(), sqlx::Error> {
    // Get or create tag
    let tag = sqlx::query!(
        "INSERT INTO tags (name) VALUES ($1) ON CONFLICT (name) DO UPDATE SET name = $1 RETURNING id",
        tag_name
    )
    .fetch_one(pool)
    .await?;

    // Link tag to post
    sqlx::query!(
        "INSERT INTO post_tags (post_id, tag_id) VALUES ($1, $2) ON CONFLICT DO NOTHING",
        post_id,
        tag.id
    )
    .execute(pool)
    .await?;

    Ok(())
}
```

### Get Posts by Tag

```rust
async fn get_posts_by_tag(
    pool: &PgPool,
    tag_name: &str,
) -> Result<Vec<Post>, sqlx::Error> {
    let posts = sqlx::query_as!(
        Post,
        r#"
        SELECT p.*
        FROM posts p
        INNER JOIN post_tags pt ON p.id = pt.post_id
        INNER JOIN tags t ON pt.tag_id = t.id
        WHERE t.name = $1
        ORDER BY p.created_at DESC
        "#,
        tag_name
    )
    .fetch_all(pool)
    .await?;

    Ok(posts)
}
```

## Transactions for Complex Operations

### Transfer Post Ownership with History

```rust
async fn transfer_post_with_history(
    pool: &PgPool,
    post_id: i32,
    new_owner_id: i32,
    transferred_by: i32,
) -> Result<(), sqlx::Error> {
    let mut tx = pool.begin().await?;

    // Verify new owner exists
    sqlx::query!("SELECT id FROM users WHERE id = $1", new_owner_id)
        .fetch_one(&mut *tx)
        .await?;

    // Get current owner
    let current = sqlx::query!("SELECT user_id FROM posts WHERE id = $1", post_id)
        .fetch_one(&mut *tx)
        .await?;

    // Update post ownership
    sqlx::query!(
        "UPDATE posts SET user_id = $1, updated_at = NOW() WHERE id = $2",
        new_owner_id,
        post_id
    )
    .execute(&mut *tx)
    .await?;

    // Record transfer in history
    sqlx::query!(
        r#"
        INSERT INTO ownership_history (post_id, from_user_id, to_user_id, transferred_by)
        VALUES ($1, $2, $3, $4)
        "#,
        post_id,
        current.user_id,
        new_owner_id,
        transferred_by
    )
    .execute(&mut *tx)
    .await?;

    tx.commit().await?;

    Ok(())
}
```

## Optimistic Locking

Prevent lost updates with version numbers:

```sql
ALTER TABLE posts ADD COLUMN version INTEGER DEFAULT 1 NOT NULL;
```

```rust
#[derive(Deserialize)]
struct UpdatePostWithVersion {
    title: Option<String>,
    content: Option<String>,
    version: i32,
}

async fn update_post_optimistic(
    pool: &PgPool,
    post_id: i32,
    update: UpdatePostWithVersion,
) -> Result<Post, ApiError> {
    let result = sqlx::query_as!(
        Post,
        r#"
        UPDATE posts
        SET
            title = COALESCE($1, title),
            content = COALESCE($2, content),
            version = version + 1,
            updated_at = NOW()
        WHERE id = $3 AND version = $4
        RETURNING *
        "#,
        update.title,
        update.content,
        post_id,
        update.version
    )
    .fetch_optional(pool)
    .await?;

    result.ok_or(ApiError::Conflict("Post was modified by another user".to_string()))
}
```

## Pagination with Total Count

```rust
#[derive(Serialize)]
struct PaginatedResponse<T> {
    items: Vec<T>,
    total: i64,
    page: i64,
    limit: i64,
    total_pages: i64,
}

async fn get_posts_paginated(
    pool: &PgPool,
    page: i64,
    limit: i64,
) -> Result<PaginatedResponse<Post>, sqlx::Error> {
    let offset = page * limit;

    // Get total count
    let total = sqlx::query!("SELECT COUNT(*) as count FROM posts")
        .fetch_one(pool)
        .await?
        .count
        .unwrap_or(0);

    // Get page of posts
    let posts = sqlx::query_as!(
        Post,
        "SELECT * FROM posts ORDER BY created_at DESC LIMIT $1 OFFSET $2",
        limit,
        offset
    )
    .fetch_all(pool)
    .await?;

    let total_pages = (total as f64 / limit as f64).ceil() as i64;

    Ok(PaginatedResponse {
        items: posts,
        total,
        page,
        limit,
        total_pages,
    })
}
```

## Database Indexes for Performance

```sql
-- Index for filtering
CREATE INDEX idx_posts_published ON posts(published);

-- Composite index for common queries
CREATE INDEX idx_posts_user_published ON posts(user_id, published);

-- Index for sorting
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);

-- Partial index (only index published posts)
CREATE INDEX idx_published_posts ON posts(created_at DESC) WHERE published = true;

-- Unique composite index
CREATE UNIQUE INDEX idx_unique_user_post_title ON posts(user_id, title);
```

## Soft Deletes

Instead of deleting, mark as deleted:

```sql
ALTER TABLE posts ADD COLUMN deleted_at TIMESTAMP NULL;

CREATE INDEX idx_posts_not_deleted ON posts(deleted_at) WHERE deleted_at IS NULL;
```

```rust
async fn soft_delete_post(
    pool: &PgPool,
    post_id: i32,
) -> Result<(), sqlx::Error> {
    sqlx::query!(
        "UPDATE posts SET deleted_at = NOW() WHERE id = $1 AND deleted_at IS NULL",
        post_id
    )
    .execute(pool)
    .await?;

    Ok(())
}

async fn list_active_posts(pool: &PgPool) -> Result<Vec<Post>, sqlx::Error> {
    let posts = sqlx::query_as!(
        Post,
        "SELECT * FROM posts WHERE deleted_at IS NULL ORDER BY created_at DESC"
    )
    .fetch_all(pool)
    .await?;

    Ok(posts)
}
```

## Practice Exercises

### Exercise 1: Popular Posts

Create a query that returns the top 10 posts by comment count.

<details>
<summary>Solution</summary>

```rust
#[derive(Serialize)]
struct PopularPost {
    id: i32,
    title: String,
    author: String,
    comment_count: i64,
}

async fn get_popular_posts(pool: &PgPool) -> Result<Vec<PopularPost>, sqlx::Error> {
    let posts = sqlx::query_as!(
        PopularPost,
        r#"
        SELECT
            p.id,
            p.title,
            u.username as author,
            COUNT(c.id) as "comment_count!"
        FROM posts p
        JOIN users u ON p.user_id = u.id
        LEFT JOIN comments c ON p.id = c.post_id
        WHERE p.published = true
        GROUP BY p.id, p.title, u.username
        ORDER BY comment_count DESC
        LIMIT 10
        "#
    )
    .fetch_all(pool)
    .await?;

    Ok(posts)
}
```
</details>

### Exercise 2: User Activity

Get users ordered by total activity (posts + comments).

<details>
<summary>Solution</summary>

```rust
#[derive(Serialize)]
struct UserActivity {
    user_id: i32,
    username: String,
    post_count: i64,
    comment_count: i64,
    total_activity: i64,
}

async fn get_user_activity(pool: &PgPool) -> Result<Vec<UserActivity>, sqlx::Error> {
    let activity = sqlx::query_as!(
        UserActivity,
        r#"
        SELECT
            u.id as user_id,
            u.username,
            COUNT(DISTINCT p.id) as "post_count!",
            COUNT(DISTINCT c.id) as "comment_count!",
            COUNT(DISTINCT p.id) + COUNT(DISTINCT c.id) as "total_activity!"
        FROM users u
        LEFT JOIN posts p ON u.id = p.user_id
        LEFT JOIN comments c ON u.id = c.user_id
        GROUP BY u.id, u.username
        ORDER BY total_activity DESC
        "#
    )
    .fetch_all(pool)
    .await?;

    Ok(activity)
}
```
</details>

### Exercise 3: Related Posts

Find posts with similar tags:

<details>
<summary>Hint</summary>

```sql
SELECT p2.*, COUNT(pt2.tag_id) as matching_tags
FROM posts p1
JOIN post_tags pt1 ON p1.id = pt1.post_id
JOIN post_tags pt2 ON pt1.tag_id = pt2.tag_id
JOIN posts p2 ON pt2.post_id = p2.id
WHERE p1.id = $1 AND p2.id != $1
GROUP BY p2.id
ORDER BY matching_tags DESC
LIMIT 5
```
</details>

## Key Takeaways

- ✅ Joins combine related data from multiple tables
- ✅ Aggregations (COUNT, SUM, AVG) provide statistics
- ✅ Full-text search enables powerful searching
- ✅ Many-to-many relationships use junction tables
- ✅ Transactions ensure data consistency
- ✅ Indexes dramatically improve query performance
- ✅ Soft deletes preserve data while hiding it
- ✅ Optimistic locking prevents lost updates

**Next**: Complete database-backed blog API practice project!

---

**Progress**: Module 13, Lesson 4 complete (66/90+ lessons total)
