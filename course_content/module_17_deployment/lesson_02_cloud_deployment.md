# Module 17: Production Deployment

# Lesson 2: Cloud Deployment

## Deploying to Fly.io

```bash
# Install flyctl
curl -L https://fly.io/install.sh | sh

# Login
flyctl auth login

# Initialize
flyctl launch

# Deploy
flyctl deploy
```

`fly.toml`:

```toml
app = "my-rust-api"

[build]
  builder = "paketobuildpacks/builder:base"

[env]
  PORT = "8080"

[[services]]
  internal_port = 8080
  protocol = "tcp"

  [[services.ports]]
    handlers = ["http"]
    port = 80

  [[services.ports]]
    handlers = ["tls", "http"]
    port = 443
```

## Railway

```bash
railway login
railway init
railway up
```

## AWS ECS / Fargate

Deploy Docker container to AWS:

```bash
# Build and push to ECR
aws ecr get-login-password | docker login --username AWS --password-stdin ECR_URL
docker build -t my-api .
docker tag my-api:latest ECR_URL/my-api:latest
docker push ECR_URL/my-api:latest

# Deploy to ECS
aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment
```

## Environment Variables

Use secrets manager:

```bash
# Set secrets on Fly.io
flyctl secrets set JWT_SECRET=your-secret DATABASE_URL=postgres://...
```

## Health Checks

```rust
async fn health_check() -> &'static str {
    "OK"
}

let app = Router::new()
    .route("/health", get(health_check))
    // ...
```

## Key Takeaways

- ✅ Fly.io for easy deployment
- ✅ Railway for quick deploys
- ✅ AWS ECS for production scale
- ✅ Use secret managers for credentials
- ✅ Implement health checks

**Next**: Monitoring and observability!

---

**Progress**: Module 17, Lesson 2 complete (80/90+ lessons total)
