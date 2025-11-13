# Module 18: Full-Stack Capstone Project

# Lesson 4: Testing, Documentation & Deployment

## Integration Tests

```rust
#[sqlx::test]
async fn test_create_and_like_post(pool: PgPool) -> Result<()> {
    let user = create_test_user(&pool).await?;
    let post = create_post(&pool, user.id, "Test post").await?;

    like_post(&pool, user.id, post.id).await?;

    let likes = get_post_likes(&pool, post.id).await?;
    assert_eq!(likes, 1);

    Ok(())
}
```

## API Documentation

Generate with `utoipa` and serve Swagger UI at `/swagger-ui`.

## Docker Deployment

```dockerfile
FROM rust:1.75 as builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim
COPY --from=builder /app/target/release/social_api /app/social_api
EXPOSE 3000
CMD ["/app/social_api"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/social
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: social
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7

volumes:
  postgres_data:
```

Deploy:

```bash
docker-compose up --build
```

## Production Checklist

- ✅ All tests passing
- ✅ Environment variables for secrets
- ✅ HTTPS enabled
- ✅ Rate limiting configured
- ✅ Monitoring and logging
- ✅ Database backups
- ✅ CI/CD pipeline
- ✅ Documentation complete

## Congratulations!

You've built a complete full-stack social platform API with:
- ✅ Authentication & authorization
- ✅ Database integration
- ✅ Real-time features
- ✅ File uploads
- ✅ Comprehensive tests
- ✅ Production deployment

**You're now a Rust full-stack developer!** 🦀🎉

---

**Progress**: Module 18 complete! (85/90+ lessons total)

**Course 100% Complete!** 🚀
