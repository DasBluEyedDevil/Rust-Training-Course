# Module 10, Lesson 2: The Crate Ecosystem — Popular Libraries & Best Practices

## Discovering Crates

**crates.io** is Rust's official package registry with over 100,000 crates.

## Essential Crates by Category

### **Serialization & Data Formats**

#### serde (1.0)
The de-facto serialization framework:

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"          # JSON
serde_yaml = "0.9"          # YAML
toml = "0.8"                # TOML
bincode = "1.3"             # Binary
```

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
struct Config {
    name: String,
    value: i32,
}

fn main() {
    let config = Config { name: "test".into(), value: 42 };

    // To JSON
    let json = serde_json::to_string(&config).unwrap();

    // From JSON
    let parsed: Config = serde_json::from_str(&json).unwrap();
}
```

### **Error Handling**

#### anyhow (1.0)
For applications (not libraries):

```toml
[dependencies]
anyhow = "1.0"
```

```rust
use anyhow::{Context, Result};

fn main() -> Result<()> {
    let config = std::fs::read_to_string("config.toml")
        .context("Failed to read config file")?;

    let parsed: Config = toml::from_str(&config)
        .context("Failed to parse config")?;

    Ok(())
}
```

#### thiserror (1.0)
For libraries (custom error types):

```toml
[dependencies]
thiserror = "1.0"
```

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum MyError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),

    #[error("Parse error at line {line}: {msg}")]
    Parse { line: usize, msg: String },

    #[error("Not found: {0}")]
    NotFound(String),
}
```

### **CLI Development**

#### clap (4.5)
Command-line argument parsing:

```toml
[dependencies]
clap = { version = "4.5", features = ["derive"] }
```

```rust
use clap::Parser;

#[derive(Parser)]
#[command(version, about, long_about = None)]
struct Args {
    #[arg(short, long)]
    name: String,

    #[arg(short, long, default_value_t = 1)]
    count: u32,
}
```

#### colored (2.1)
Terminal colors:

```rust
use colored::*;

println!("{}", "Success!".green().bold());
println!("{}", "Error!".red());
println!("{}", "Warning".yellow());
```

#### indicatif (0.17)
Progress bars:

```rust
use indicatif::ProgressBar;

let pb = ProgressBar::new(100);
for _ in 0..100 {
    pb.inc(1);
    std::thread::sleep(Duration::from_millis(50));
}
pb.finish_with_message("Done!");
```

### **Async Runtime**

#### tokio (1.0)
Async runtime for I/O-heavy applications:

```toml
[dependencies]
tokio = { version = "1.x", features = ["full"] }
```

```rust
#[tokio::main]
async fn main() {
    let result = fetch_data().await;
    println!("{:?}", result);
}

async fn fetch_data() -> Result<String, Box<dyn std::error::Error>> {
    let resp = reqwest::get("https://api.example.com/data")
        .await?
        .text()
        .await?;

    Ok(resp)
}
```

### **Web Development**

#### axum (0.8.x)
Modern web framework:

```toml
[dependencies]
axum = "0.8.x"
tokio = { version = "1.x", features = ["full"] }
```

```rust
use axum::{routing::get, Router};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(|| async { "Hello, World!" }));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    axum::serve(listener, app)
        .await
        .unwrap();
}
```

#### actix-web (4.0)
High-performance web framework:

```toml
[dependencies]
actix-web = "4"
```

### **HTTP Clients**

#### reqwest (0.11)
Easy HTTP client:

```toml
[dependencies]
reqwest = { version = "0.11", features = ["json"] }
```

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let response = reqwest::get("https://api.github.com/repos/rust-lang/rust")
        .await?
        .json::<serde_json::Value>()
        .await?;

    println!("{:#?}", response);
    Ok(())
}
```

### **Database Access**

#### sqlx (0.8.x)
Async SQL toolkit:

```toml
[dependencies]
sqlx = { version = "0.8.x", features = ["runtime-tokio-native-tls", "sqlite"] }
```

```rust
use sqlx::sqlite::SqlitePool;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = SqlitePool::connect("sqlite:db.sqlite3").await?;

    let row: (i64,) = sqlx::query_as("SELECT COUNT(*) FROM users")
        .fetch_one(&pool)
        .await?;

    println!("Users: {}", row.0);
    Ok(())
}
```

### **Date & Time**

#### chrono (0.4)
Date and time handling:

```rust
use chrono::{DateTime, Local, Utc};

fn main() {
    let now = Local::now();
    println!("{}", now.format("%Y-%m-%d %H:%M:%S"));

    let utc: DateTime<Utc> = Utc::now();
    println!("{}", utc.to_rfc3339());
}
```

### **Logging**

#### tracing (0.1)
Structured logging:

```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = "0.3"
```

```rust
use tracing::{info, warn, error, debug};

fn main() {
    tracing_subscriber::fmt::init();

    info!("Application started");
    warn!("This is a warning");
    error!("An error occurred");
    debug!(user_id = 123, "Processing request");
}
```

### **Testing**

#### proptest (1.4)
Property-based testing:

```toml
[dev-dependencies]
proptest = "1.4"
```

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_reverse_reverse(s in ".*") {
        let reversed: String = s.chars().rev().collect();
        let double_reversed: String = reversed.chars().rev().collect();
        assert_eq!(s, double_reversed);
    }
}
```

## Evaluating Crates

### **Quality Indicators**

1. **Download count**: High downloads indicate popularity
2. **Recent updates**: Active maintenance
3. **Documentation**: Well-documented API
4. **Tests**: Comprehensive test coverage
5. **Dependencies**: Minimal, well-chosen dependencies
6. **Community**: GitHub stars, issues, PRs

### **Check Before Using**

```bash
# View crate information
cargo info serde

# Check for security vulnerabilities
cargo audit

# View dependency tree
cargo tree

# Check license compatibility
cargo license
```

## Publishing Your Own Crate

### **Step 1: Prepare**

```toml
[package]
name = "my_awesome_crate"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A brief description"
license = "MIT OR Apache-2.0"
readme = "README.md"
repository = "https://github.com/username/repo"
documentation = "https://docs.rs/my_awesome_crate"
keywords = ["keyword1", "keyword2"]
categories = ["category"]
```

### **Step 2: Test & Document**

```bash
# Run all tests
cargo test

# Check documentation
cargo doc --open

# Check for issues
cargo clippy
```

### **Step 3: Publish**

```bash
# Login to crates.io
cargo login YOUR_API_TOKEN

# Dry run
cargo publish --dry-run

# Publish!
cargo publish
```

## Best Practices

### **Dependency Management**

✅ **DO:**
- Use specific version ranges
- Minimize dependencies
- Use `cargo update` carefully
- Lock versions with `Cargo.lock` (commit for binaries)

❌ **DON'T:**
- Use `*` for versions
- Include unnecessary dependencies
- Ignore security advisories

### **Feature Flags**

```toml
[features]
default = ["std"]
std = []
full = ["serde", "chrono"]

[dependencies]
serde = { version = "1.0", optional = true }
chrono = { version = "0.4", optional = true }
```

### **Semantic Versioning**

- `0.1.0` → `0.1.1`: Bug fixes
- `0.1.0` → `0.2.0`: New features
- `0.1.0` → `1.0.0`: Breaking changes

## Quick Reference: Must-Know Crates

| Category | Crate | Purpose |
|----------|-------|---------|
| Serialization | serde | Data serialization |
| JSON | serde_json | JSON support |
| Error handling | anyhow | Application errors |
| Error handling | thiserror | Library errors |
| CLI | clap | Argument parsing |
| Async runtime | tokio | Async I/O |
| HTTP client | reqwest | HTTP requests |
| Web framework | axum | Web applications |
| Database | sqlx | SQL databases |
| Logging | tracing | Structured logging |
| Testing | proptest | Property testing |
| Date/Time | chrono | Date handling |

## Key Takeaways

- ✅ crates.io is Rust's package registry
- ✅ Evaluate crates by downloads, maintenance, docs, tests
- ✅ Use serde for serialization, anyhow/thiserror for errors
- ✅ clap for CLI, tokio for async, axum/actix for web
- ✅ Check security with `cargo audit`
- ✅ Follow semantic versioning
- ✅ Document and test before publishing

**Next**: Module 11 - Next Steps & Advanced Topics!

---

**Progress**: Module 10, Lesson 2 complete (56/60 lessons total)
