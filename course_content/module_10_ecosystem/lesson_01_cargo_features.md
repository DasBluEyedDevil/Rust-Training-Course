# Module 10: The Rust Ecosystem

# Lesson 1: Mastering Cargo — Features, Workspaces & More

## Understanding Cargo.toml

Cargo.toml is the heart of every Rust project. Let's explore its full capabilities.

## Basic Structure

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A brief description"
license = "MIT"
repository = "https://github.com/username/project"
keywords = ["cli", "tool"]
categories = ["command-line-utilities"]

[dependencies]
# Regular dependencies

[dev-dependencies]
# Test-only dependencies

[build-dependencies]
# Build script dependencies
```

## Dependency Specifications

### **Version Requirements**

```toml
[dependencies]
# Exact version
serde = "=1.0.0"

# Caret (compatible updates)
serde = "^1.0"    # >= 1.0.0, < 2.0.0
serde = "1.0"     # Same as ^1.0

# Tilde (minor updates only)
serde = "~1.2"    # >= 1.2.0, < 1.3.0

# Wildcard
serde = "1.*"     # >= 1.0.0, < 2.0.0

# Greater/less than
serde = ">= 1.0, < 2.0"

# Latest
serde = "*"       # Not recommended!
```

### **Git Dependencies**

```toml
[dependencies]
my_crate = { git = "https://github.com/user/repo" }

# Specific branch
my_crate = { git = "https://github.com/user/repo", branch = "develop" }

# Specific tag
my_crate = { git = "https://github.com/user/repo", tag = "v1.0.0" }

# Specific commit
my_crate = { git = "https://github.com/user/repo", rev = "abc123" }
```

### **Path Dependencies**

```toml
[dependencies]
my_local_crate = { path = "../my_local_crate" }
```

## Features

Features enable conditional compilation:

```toml
[features]
default = ["std"]  # Enabled by default
std = []
serde = ["dep:serde", "dep:serde_json"]
experimental = []

[dependencies]
serde = { version = "1.0", optional = true }
serde_json = { version = "1.0", optional = true }
```

**Usage:**

```bash
# Build with specific features
cargo build --features serde
cargo build --features "serde experimental"

# Build without default features
cargo build --no-default-features

# Build with all features
cargo build --all-features
```

**In code:**

```rust
#[cfg(feature = "serde")]
use serde::{Serialize, Deserialize};

#[cfg(feature = "serde")]
#[derive(Serialize, Deserialize)]
pub struct Data {
    value: i32,
}

#[cfg(not(feature = "serde"))]
pub struct Data {
    value: i32,
}
```

## Workspaces

Manage multiple related packages:

### **Workspace Cargo.toml (root)**

```toml
[workspace]
members = [
    "crates/core",
    "crates/cli",
    "crates/server",
]

[workspace.dependencies]
# Shared dependency versions
serde = "1.0"
tokio = "1.0"
```

### **Member Cargo.toml**

```toml
[package]
name = "my-cli"
version = "0.1.0"
edition = "2021"

[dependencies]
# Reference workspace package
my-core = { path = "../core" }

# Use workspace dependency version
serde = { workspace = true }
```

**Benefits:**
- Shared `target/` directory
- Unified dependency versions
- Build all packages: `cargo build --workspace`
- Test all packages: `cargo test --workspace`

## Profiles

Optimize builds for different scenarios:

```toml
[profile.dev]
opt-level = 0      # No optimization
debug = true       # Include debug info

[profile.release]
opt-level = 3      # Maximum optimization
debug = false      # No debug info
lto = true         # Link-time optimization
codegen-units = 1  # Better optimization, slower compile

[profile.release-with-debug]
inherits = "release"
debug = true       # Release optimizations + debug symbols
```

**Usage:**

```bash
cargo build --release
cargo build --profile release-with-debug
```

## Build Scripts

`build.rs` runs before compilation:

```rust
// build.rs
fn main() {
    println!("cargo:rerun-if-changed=build.rs");

    // Set environment variable
    println!("cargo:rustc-env=BUILD_TIME={}",
        chrono::Local::now().format("%Y-%m-%d"));

    // Link library
    println!("cargo:rustc-link-lib=mylib");

    // Add compiler flag
    println!("cargo:rustc-cfg=has_feature");
}
```

**In Cargo.toml:**

```toml
[build-dependencies]
chrono = "0.4"
```

## Useful Cargo Commands

```bash
# Check project (fast, no binary)
cargo check

# Build documentation
cargo doc --open

# Run specific binary
cargo run --bin my_app

# Run specific example
cargo run --example my_example

# Show dependency tree
cargo tree

# Update dependencies
cargo update

# Clean build artifacts
cargo clean

# Format code
cargo fmt

# Lint code
cargo clippy

# Benchmark
cargo bench

# Show outdated dependencies
cargo outdated  # Requires cargo-outdated plugin
```

## Cargo Plugins

Install useful plugins:

```bash
# Audit dependencies for security vulnerabilities
cargo install cargo-audit
cargo audit

# Check for outdated dependencies
cargo install cargo-outdated
cargo outdated

# Generate license file
cargo install cargo-license
cargo license

# Expand macros
cargo install cargo-expand
cargo expand

# Watch for changes and rebuild
cargo install cargo-watch
cargo watch -x run

# Binary size optimization
cargo install cargo-bloat
cargo bloat --release
```

## Example: Complete Cargo.toml

```toml
[package]
name = "taskmaster"
version = "1.0.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A powerful task management CLI"
license = "MIT OR Apache-2.0"
repository = "https://github.com/username/taskmaster"
readme = "README.md"
keywords = ["cli", "task", "productivity"]
categories = ["command-line-utilities"]

[features]
default = ["cli"]
cli = ["clap"]
web = ["actix-web"]
all = ["cli", "web"]

[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
chrono = { version = "0.4", features = ["serde"] }
clap = { version = "4.5", features = ["derive"], optional = true }
actix-web = { version = "4", optional = true }

[dev-dependencies]
tempfile = "3"

[profile.release]
lto = true
codegen-units = 1
strip = true        # Remove debug symbols

[profile.dev]
opt-level = 1       # Some optimization for faster dev builds

[[bin]]
name = "taskmaster"
path = "src/main.rs"

[[example]]
name = "api_demo"
required-features = ["web"]
```

## Key Takeaways

- ✅ Cargo.toml controls dependencies, features, and build settings
- ✅ Version specifications: exact (=), caret (^), tilde (~)
- ✅ Features enable conditional compilation
- ✅ Workspaces manage multiple related packages
- ✅ Profiles optimize builds for different scenarios
- ✅ Build scripts enable custom build steps
- ✅ Cargo plugins extend functionality

**Next**: Exploring crates.io and discovering popular crates!

---

**Progress**: Module 10, Lesson 1 complete (55/60 lessons total)
