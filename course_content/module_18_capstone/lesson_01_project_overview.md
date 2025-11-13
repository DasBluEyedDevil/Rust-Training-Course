# Module 18: Full-Stack Capstone Project

# Lesson 1: Project Overview - Social Platform API

## What We're Building

A complete **social platform API** with:
- User authentication (JWT)
- User profiles
- Posts with likes and comments
- Follow/following system
- Feed algorithm
- Real-time notifications (WebSockets)
- File uploads (profile pictures)
- Full-text search
- Rate limiting
- Comprehensive tests
- Docker deployment

## Tech Stack

**Backend:**
- Rust + Axum 0.8
- PostgreSQL + SQLx 0.8
- JWT authentication
- WebSockets for real-time
- Redis for caching

**Features:**
- RESTful API
- Role-based access control
- Database migrations
- API documentation (Swagger)
- Monitoring and logging

## Database Schema

```sql
-- Users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    bio TEXT,
    avatar_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Posts
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    content TEXT NOT NULL,
    image_url VARCHAR(500),
    likes_count INTEGER DEFAULT 0,
    comments_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Comments
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    user_id INTEGER REFERENCES users(id),
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Likes
CREATE TABLE post_likes (
    user_id INTEGER REFERENCES users(id),
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (user_id, post_id)
);

-- Follows
CREATE TABLE follows (
    follower_id INTEGER REFERENCES users(id),
    following_id INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (follower_id, following_id),
    CHECK (follower_id != following_id)
);
```

## API Endpoints

```
POST   /auth/register
POST   /auth/login
POST   /auth/logout

GET    /users/{username}
PUT    /users/me
GET    /users/{username}/followers
GET    /users/{username}/following
POST   /users/{username}/follow
DELETE /users/{username}/follow

GET    /posts
GET    /posts/feed
GET    /posts/{id}
POST   /posts
PUT    /posts/{id}
DELETE /posts/{id}
POST   /posts/{id}/like
DELETE /posts/{id}/like

GET    /posts/{id}/comments
POST   /posts/{id}/comments
DELETE /comments/{id}

GET    /search?q=...

WS     /ws/notifications
```

## Project Structure

```
social_api/
├── src/
│   ├── main.rs
│   ├── config.rs
│   ├── models/
│   ├── handlers/
│   ├── middleware/
│   ├── services/
│   └── utils/
├── migrations/
├── tests/
├── Dockerfile
├── docker-compose.yml
└── README.md
```

## Next Steps

1. Set up project structure
2. Implement authentication
3. Build core features (posts, comments, likes)
4. Add social features (follows, feed)
5. Implement real-time notifications
6. Add file uploads
7. Write tests
8. Deploy to production

**Next**: Implementing the authentication system!

---

**Progress**: Module 18, Lesson 1 complete (82/90+ lessons total)
