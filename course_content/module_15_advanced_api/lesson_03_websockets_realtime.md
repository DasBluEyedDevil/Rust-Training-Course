# Module 15: Advanced API Development

# Lesson 3: WebSockets & Real-Time Communication

## What are WebSockets?

Bi-directional, persistent connections for real-time data.

### Setup

```toml
[dependencies]
axum = { version = "0.8", features = ["ws"] }
```

### Basic WebSocket Handler

```rust
use axum::{
    extract::ws::{WebSocket, WebSocketUpgrade},
    response::Response,
};

async fn ws_handler(ws: WebSocketUpgrade) -> Response {
    ws.on_upgrade(handle_socket)
}

async fn handle_socket(mut socket: WebSocket) {
    while let Some(msg) = socket.recv().await {
        if let Ok(msg) = msg {
            // Echo back
            if socket.send(msg).await.is_err() {
                break;
            }
        }
    }
}

let app = Router::new()
    .route("/ws", get(ws_handler));
```

### Chat Room Example

```rust
use std::sync::Arc;
use tokio::sync::broadcast;

type Tx = broadcast::Sender<String>;

async fn chat_handler(
    ws: WebSocketUpgrade,
    State(tx): State<Arc<Tx>>,
) -> Response {
    ws.on_upgrade(|socket| handle_chat(socket, tx))
}

async fn handle_chat(socket: WebSocket, tx: Arc<Tx>) {
    let (mut sender, mut receiver) = socket.split();
    let mut rx = tx.subscribe();

    // Send messages from broadcast to client
    let mut send_task = tokio::spawn(async move {
        while let Ok(msg) = rx.recv().await {
            if sender.send(Message::Text(msg)).await.is_err() {
                break;
            }
        }
    });

    // Receive from client and broadcast
    let tx2 = tx.clone();
    let mut recv_task = tokio::spawn(async move {
        while let Some(Ok(Message::Text(text))) = receiver.next().await {
            let _ = tx2.send(text);
        }
    });

    // Wait for either task to finish
    tokio::select! {
        _ = (&mut send_task) => recv_task.abort(),
        _ = (&mut recv_task) => send_task.abort(),
    }
}

#[tokio::main]
async fn main() {
    let (tx, _rx) = broadcast::channel(100);
    let app_state = Arc::new(tx);

    let app = Router::new()
        .route("/ws", get(chat_handler))
        .with_state(app_state);

    // ...
}
```

### Client Example (JavaScript)

```javascript
const ws = new WebSocket('ws://localhost:3000/ws');

ws.onmessage = (event) => {
    console.log('Received:', event.data);
};

ws.send('Hello from client!');
```

## Server-Sent Events (SSE)

One-way streaming from server to client:

```rust
use axum::{
    response::sse::{Event, Sse},
    response::IntoResponse,
};
use tokio_stream::StreamExt as _;
use std::convert::Infallible;

async fn sse_handler() -> Sse<impl Stream<Item = Result<Event, Infallible>>> {
    let stream = tokio_stream::iter(0..)
        .throttle(Duration::from_secs(1))
        .map(|i| Ok(Event::default().data(format!("Event {}", i))));

    Sse::new(stream)
}
```

## Key Takeaways

- ✅ WebSockets for bi-directional real-time
- ✅ SSE for server-to-client streaming
- ✅ Broadcast channels for chat/notifications
- ✅ Use tokio for async handling

**Module 15 Complete!**

---

**Progress**: Module 15 complete! (75/90+ lessons total)
