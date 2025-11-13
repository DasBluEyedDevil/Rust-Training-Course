# Module 17: Production Deployment

# Lesson 1: Docker & Containerization

## Dockerfile for Rust

```dockerfile
# Build stage
FROM rust:1.75 as builder

WORKDIR /app
COPY . .

# Build release binary
RUN cargo build --release

# Runtime stage
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY --from=builder /app/target/release/my_api /app/my_api

EXPOSE 3000

CMD ["./my_api"]
```

## Docker Compose

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Build and run:

```bash
docker-compose up --build
```

## Multi-stage Optimization

```dockerfile
# Use Alpine for smaller image
FROM rust:1.75-alpine as builder

RUN apk add --no-cache musl-dev

WORKDIR /app
COPY Cargo.* ./
COPY src ./src

RUN cargo build --release --target x86_64-unknown-linux-musl

FROM alpine:latest

COPY --from=builder /app/target/x86_64-unknown-linux-musl/release/my_api /app/my_api

EXPOSE 3000
CMD ["/app/my_api"]
```

## Key Takeaways

- ✅ Multi-stage builds reduce image size
- ✅ Docker Compose for local development
- ✅ Use Alpine for minimal images
- ✅ Separate build and runtime stages

**Next**: Deployment to cloud platforms!

---

**Progress**: Module 17, Lesson 1 complete (79/90+ lessons total)
