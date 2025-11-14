# Module 11: Next Steps & Async Introduction

# Lesson 1: Hands-On Async/Await Practice

## Why Async Programming?

You've built command-line applications that run **synchronously** - each operation waits for the previous one to finish.

**Synchronous (blocking):**
```rust
fn main() {
    let file1 = read_file("data1.txt");  // Wait for this...
    let file2 = read_file("data2.txt");  // Then this...
    let file3 = read_file("data3.txt");  // Then this...
    // Total time: ~3 seconds if each takes 1 second
}
```

**Asynchronous (non-blocking):**
```rust
#[tokio::main]
async fn main() {
    // Start all three at once!
    let (file1, file2, file3) = tokio::join!(
        read_file_async("data1.txt"),
        read_file_async("data2.txt"),
        read_file_async("data3.txt"),
    );
    // Total time: ~1 second (all run concurrently)
}
```

**Async is essential for:**
- Web servers (handle many requests simultaneously)
- Network I/O (don't wait for slow network calls)
- Database queries (run multiple queries in parallel)
- File I/O in high-performance applications

## Setup

```toml
# Cargo.toml
[package]
name = "async_practice"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { version = "1.x", features = ["full"] }
reqwest = "0.12"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

## Understanding async/await

### The Basics

**async fn** - Returns a **Future** instead of executing immediately:

```rust
// This doesn't execute yet!
async fn fetch_data() -> String {
    "Hello from async".to_string()
}

// To actually run it:
#[tokio::main]
async fn main() {
    let result = fetch_data().await;  // .await executes the future
    println!("{}", result);
}
```

**Key concepts:**
1. `async fn` creates a **Future** (a value that will be ready later)
2. `.await` **executes** the future and waits for the result
3. `#[tokio::main]` sets up the **async runtime** that manages all futures

### Example: Download a File

**Synchronous (blocking):**
```rust
use std::fs;

fn download_blocking() -> Result<String, Box<dyn std::error::Error>> {
    // This blocks the entire program until download completes!
    let response = reqwest::blocking::get("https://api.github.com/users/rust-lang")?
        .text()?;

    fs::write("rust-lang.json", &response)?;
    Ok(response)
}
```

**Asynchronous (non-blocking):**
```rust
use tokio::fs;

async fn download_async() -> Result<String, Box<dyn std::error::Error>> {
    // Other async tasks can run while waiting for network!
    let response = reqwest::get("https://api.github.com/users/rust-lang")
        .await?      // Wait for request
        .text()
        .await?;     // Wait for body

    fs::write("rust-lang.json", &response).await?;  // Wait for file write
    Ok(response)
}
```

## Practice 1: Your First Async Program

Create `src/main.rs`:

```rust
use tokio::time::{sleep, Duration};

async fn task_one() {
    println!("Task 1: Started");
    sleep(Duration::from_secs(2)).await;  // Simulate work
    println!("Task 1: Finished");
}

async fn task_two() {
    println!("Task 2: Started");
    sleep(Duration::from_secs(1)).await;
    println!("Task 2: Finished");
}

#[tokio::main]
async fn main() {
    println!("=== Running Sequentially ===");
    task_one().await;
    task_two().await;
    // Total time: 3 seconds

    println!("\n=== Running Concurrently ===");
    // Start both at the same time!
    tokio::join!(task_one(), task_two());
    // Total time: 2 seconds (limited by slower task)
}
```

**Run it:**
```bash
cargo run
```

**Output:**
```
=== Running Sequentially ===
Task 1: Started
Task 1: Finished
Task 2: Started
Task 2: Finished

=== Running Concurrently ===
Task 1: Started
Task 2: Started
Task 2: Finished
Task 1: Finished
```

Notice Task 2 finishes before Task 1 when run concurrently!

## Practice 2: Fetching Multiple URLs

```rust
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct GitHubUser {
    login: String,
    name: Option<String>,
    public_repos: u32,
}

async fn fetch_user(username: &str) -> Result<GitHubUser, reqwest::Error> {
    let url = format!("https://api.github.com/users/{}", username);

    reqwest::get(&url)
        .await?
        .json::<GitHubUser>()
        .await
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let usernames = vec!["torvalds", "graydon", "BurntSushi"];

    println!("Fetching users sequentially...");
    let start = std::time::Instant::now();

    for username in &usernames {
        match fetch_user(username).await {
            Ok(user) => println!("{}: {} repos", user.login, user.public_repos),
            Err(e) => eprintln!("Error fetching {}: {}", username, e),
        }
    }

    println!("Sequential time: {:?}\n", start.elapsed());

    // Now concurrently!
    println!("Fetching users concurrently...");
    let start = std::time::Instant::now();

    let tasks = usernames.iter().map(|username| fetch_user(username));
    let results = futures::future::join_all(tasks).await;

    for result in results {
        match result {
            Ok(user) => println!("{}: {} repos", user.login, user.public_repos),
            Err(e) => eprintln!("Error: {}", e),
        }
    }

    println!("Concurrent time: {:?}", start.elapsed());

    Ok(())
}
```

Add to `Cargo.toml`:
```toml
[dependencies]
futures = "0.3"
```

## Practice 3: tokio::spawn for Background Tasks

`tokio::spawn` runs a task in the background:

```rust
use tokio::time::{sleep, Duration};

async fn background_worker(id: u32) {
    for i in 1..=3 {
        println!("Worker {}: Step {}", id, i);
        sleep(Duration::from_millis(500)).await;
    }
}

#[tokio::main]
async fn main() {
    println!("Starting workers...\n");

    // Spawn 3 background tasks
    let worker1 = tokio::spawn(background_worker(1));
    let worker2 = tokio::spawn(background_worker(2));
    let worker3 = tokio::spawn(background_worker(3));

    // Do other work while they run
    println!("Main: Doing other work...");
    sleep(Duration::from_secs(1)).await;
    println!("Main: Done with work\n");

    // Wait for all workers to finish
    let _ = tokio::join!(worker1, worker2, worker3);

    println!("\nAll workers finished!");
}
```

## Practice 4: Returning Values from Spawned Tasks

```rust
async fn calculate(x: i32) -> i32 {
    tokio::time::sleep(Duration::from_secs(1)).await;
    x * 2
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let handle1 = tokio::spawn(calculate(5));
    let handle2 = tokio::spawn(calculate(10));
    let handle3 = tokio::spawn(calculate(15));

    // Join handles return Result<T, JoinError>
    let result1 = handle1.await?;
    let result2 = handle2.await?;
    let result3 = handle3.await?;

    println!("Results: {}, {}, {}", result1, result2, result3);
    // Results: 10, 20, 30

    Ok(())
}
```

## Practice 5: Error Handling with async

```rust
use reqwest::Error;

async fn fetch_url(url: &str) -> Result<String, Error> {
    reqwest::get(url)
        .await?
        .text()
        .await
}

#[tokio::main]
async fn main() {
    let urls = vec![
        "https://api.github.com/users/rust-lang",
        "https://invalid-url-that-will-fail.com/api",
        "https://api.github.com/users/tokio-rs",
    ];

    for url in urls {
        match fetch_url(url).await {
            Ok(body) => println!("✅ Fetched {} bytes from {}", body.len(), url),
            Err(e) => println!("❌ Error fetching {}: {}", url, e),
        }
    }
}
```

## Practice 6: Real-World Example - Parallel File Downloads

```rust
use tokio::fs::File;
use tokio::io::AsyncWriteExt;

async fn download_file(url: &str, filename: &str) -> Result<(), Box<dyn std::error::Error>> {
    println!("⬇️  Downloading {} ...", url);

    let response = reqwest::get(url).await?;
    let bytes = response.bytes().await?;

    let mut file = File::create(filename).await?;
    file.write_all(&bytes).await?;

    println!("✅ Saved to {}", filename);

    Ok(())
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let downloads = vec![
        ("https://www.rust-lang.org/", "rust-homepage.html"),
        ("https://doc.rust-lang.org/book/", "rust-book.html"),
        ("https://crates.io/", "crates-io.html"),
    ];

    let tasks = downloads.into_iter().map(|(url, filename)| {
        tokio::spawn(async move {
            if let Err(e) = download_file(url, filename).await {
                eprintln!("❌ Error: {}", e);
            }
        })
    });

    // Wait for all downloads to complete
    for task in tasks {
        task.await?;
    }

    println!("\n🎉 All downloads complete!");

    Ok(())
}
```

## Understanding #[tokio::main]

This macro:

```rust
#[tokio::main]
async fn main() {
    // Your async code
}
```

Expands to:

```rust
fn main() {
    tokio::runtime::Runtime::new()
        .unwrap()
        .block_on(async {
            // Your async code
        })
}
```

The **runtime** manages all async tasks:
- Schedules tasks
- Manages threads
- Handles I/O
- Wakes tasks when ready

## Key Differences: sync vs async

| **Synchronous** | **Asynchronous** |
|----------------|------------------|
| `fn` | `async fn` |
| Returns `T` | Returns `Future<Output = T>` |
| Executes immediately | Executes when `.await`ed |
| Blocks thread | Yields to other tasks |
| `std::fs::read` | `tokio::fs::read` |
| `std::thread::sleep` | `tokio::time::sleep` |
| `reqwest::blocking::get` | `reqwest::get` |

## Practice Exercise: Build a URL Checker

Create a program that checks if multiple URLs are online:

**Requirements:**
1. Accept a list of URLs
2. Check each URL concurrently using `reqwest`
3. Report which are online (status 200) and which are down
4. Print total time taken

**Starter code:**

```rust
use reqwest;
use tokio::time::Instant;

async fn check_url(url: &str) -> Result<bool, reqwest::Error> {
    // TODO: Make GET request and check if status is 200
    // Return Ok(true) if online, Ok(false) if not
    todo!()
}

#[tokio::main]
async fn main() {
    let urls = vec![
        "https://www.rust-lang.org",
        "https://github.com",
        "https://this-site-does-not-exist-12345.com",
        "https://crates.io",
    ];

    let start = Instant::now();

    // TODO: Check all URLs concurrently
    // Print status for each

    println!("\nTotal time: {:?}", start.elapsed());
}
```

**Solution:**

```rust
use reqwest;
use tokio::time::Instant;

async fn check_url(url: &str) -> Result<bool, reqwest::Error> {
    match reqwest::get(url).await {
        Ok(response) => Ok(response.status().is_success()),
        Err(_) => Ok(false),
    }
}

#[tokio::main]
async fn main() {
    let urls = vec![
        "https://www.rust-lang.org",
        "https://github.com",
        "https://this-site-does-not-exist-12345.com",
        "https://crates.io",
    ];

    let start = Instant::now();

    let tasks = urls.iter().map(|url| {
        let url = url.to_string();
        tokio::spawn(async move {
            match check_url(&url).await {
                Ok(true) => println!("✅ {} is online", url),
                Ok(false) => println!("❌ {} is down", url),
                Err(e) => println!("⚠️  {} error: {}", url, e),
            }
        })
    });

    for task in tasks {
        let _ = task.await;
    }

    println!("\nTotal time: {:?}", start.elapsed());
}
```

## Common Patterns

### Pattern 1: Join All

```rust
// Wait for all to complete
let results = tokio::join!(task1(), task2(), task3());
```

### Pattern 2: Select First

```rust
// Return as soon as ONE completes
let result = tokio::select! {
    res1 = task1() => res1,
    res2 = task2() => res2,
};
```

### Pattern 3: Timeout

```rust
use tokio::time::{timeout, Duration};

match timeout(Duration::from_secs(5), slow_operation()).await {
    Ok(result) => println!("Completed: {:?}", result),
    Err(_) => println!("Timed out!"),
}
```

## When to Use Async

✅ **Use async when:**
- Building web servers
- Making network requests
- Reading/writing files in high-performance apps
- Database queries
- WebSockets or real-time communication

❌ **Don't use async for:**
- CPU-intensive work (use `std::thread` instead)
- Simple CLI tools with no I/O
- When you don't need concurrency

## Key Takeaways

- ✅ `async fn` creates a **Future** that must be `.await`ed to execute
- ✅ `#[tokio::main]` sets up the async **runtime**
- ✅ Use `tokio::spawn` to run tasks in the **background**
- ✅ `tokio::join!` runs multiple tasks **concurrently** and waits for all
- ✅ Async is for **I/O-bound** tasks, not CPU-bound
- ✅ Use async versions of libraries (`tokio::fs`, `reqwest`, etc.)
- ✅ Error handling works the same with `Result` and `?`

## What's Next?

Now you understand async/await! In the next modules, you'll use these concepts to:
- Build web servers with Axum (Module 12)
- Query databases with SQLx (Module 13)
- Handle WebSockets for real-time features (Module 15)

**Next**: Advanced topics overview and career paths!

---

**Progress**: Module 11, Lesson 1 complete (68/85 lessons total)
