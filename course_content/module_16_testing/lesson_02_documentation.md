# Module 16: Testing & Quality

# Lesson 2: Documentation & Code Quality

## Doc Comments

```rust
/// Creates a new user in the database.
///
/// # Arguments
///
/// * `username` - The user's username
/// * `email` - The user's email address
///
/// # Example
///
/// ```
/// let user = create_user("alice", "alice@example.com").await?;
/// ```
///
/// # Errors
///
/// Returns an error if the username or email already exists.
pub async fn create_user(username: &str, email: &str) -> Result<User, ApiError> {
    // ...
}
```

Generate docs:

```bash
cargo doc --open
```

## Clippy (Linter)

```bash
cargo clippy

# Fix automatically
cargo clippy --fix
```

## Formatting

```bash
cargo fmt

# Check formatting
cargo fmt -- --check
```

## Pre-commit Hooks

Create `.git/hooks/pre-commit`:

```bash
#!/bin/sh
cargo fmt -- --check
cargo clippy -- -D warnings
cargo test
```

## Key Takeaways

- ✅ Document public APIs
- ✅ Run `cargo doc` for HTML docs
- ✅ Use clippy for linting
- ✅ Format with `cargo fmt`
- ✅ Set up pre-commit hooks

**Next**: Benchmarking and performance!

---

**Progress**: Module 16, Lesson 2 complete (77/90+ lessons total)
