# Module 16: Testing & Quality

# Lesson 3: Performance & Benchmarking

## Benchmarking with Criterion

```toml
[dev-dependencies]
criterion = "0.5"

[[bench]]
name = "my_benchmark"
harness = false
```

`benches/my_benchmark.rs`:

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 1,
        1 => 1,
        n => fibonacci(n-1) + fibonacci(n-2),
    }
}

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("fib 20", |b| b.iter(|| fibonacci(black_box(20))));
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
```

Run:

```bash
cargo bench
```

## Database Connection Pooling

```rust
let pool = PgPoolOptions::new()
    .max_connections(20)       // Increase for high traffic
    .min_connections(5)        // Keep some connections warm
    .acquire_timeout(Duration::from_secs(3))
    .connect(&database_url)
    .await?;
```

## Caching

```rust
use std::sync::Arc;
use tokio::sync::RwLock;
use std::collections::HashMap;

#[derive(Clone)]
struct Cache {
    data: Arc<RwLock<HashMap<String, String>>>,
}

impl Cache {
    async fn get(&self, key: &str) -> Option<String> {
        self.data.read().await.get(key).cloned()
    }

    async fn set(&self, key: String, value: String) {
        self.data.write().await.insert(key, value);
    }
}
```

## Profiling

```bash
# Install flamegraph
cargo install flamegraph

# Generate flamegraph
cargo flamegraph --bin my_api
```

## Key Takeaways

- ✅ Benchmark with criterion
- ✅ Optimize database connection pools
- ✅ Use caching for expensive operations
- ✅ Profile with flamegraph

**Module 16 Complete!**

---

**Progress**: Module 16 complete! (78/90+ lessons total)
