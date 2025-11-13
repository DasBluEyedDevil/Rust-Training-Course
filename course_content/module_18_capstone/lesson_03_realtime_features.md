# Module 18: Full-Stack Capstone Project

# Lesson 3: Real-Time Features & File Uploads

## WebSocket Notifications

```rust
use axum::extract::ws::{WebSocket, WebSocketUpgrade};
use tokio::sync::broadcast;

type NotificationChannel = broadcast::Sender<Notification>;

async fn ws_notifications(
    ws: WebSocketUpgrade,
    State(tx): State<Arc<NotificationChannel>>,
    user: CurrentUser,
) -> Response {
    ws.on_upgrade(|socket| handle_notifications(socket, tx, user))
}

async fn handle_notifications(
    socket: WebSocket,
    tx: Arc<NotificationChannel>,
    user: CurrentUser,
) {
    let (mut sender, _) = socket.split();
    let mut rx = tx.subscribe();

    while let Ok(notification) = rx.recv().await {
        if notification.user_id == user.user_id {
            let msg = serde_json::to_string(&notification).unwrap();
            if sender.send(Message::Text(msg)).await.is_err() {
                break;
            }
        }
    }
}
```

## File Uploads

```toml
[dependencies]
tower-http = { version = "0.5", features = ["fs"] }
multer = "3.0"
```

```rust
use axum::extract::Multipart;

async fn upload_avatar(
    user: CurrentUser,
    mut multipart: Multipart,
) -> Result<Json<UploadResponse>, ApiError> {
    while let Some(field) = multipart.next_field().await? {
        if field.name() == Some("file") {
            let data = field.bytes().await?;
            let filename = format!("avatars/{}.jpg", user.user_id);

            tokio::fs::write(&filename, data).await?;

            return Ok(Json(UploadResponse {
                url: format!("/uploads/{}", filename),
            }));
        }
    }

    Err(ApiError::BadRequest("No file uploaded".into()))
}
```

**Key Takeaways:**
- ✅ WebSocket notifications
- ✅ File upload handling
- ✅ Broadcast to connected clients

**Next**: Testing and deployment!

---

**Progress**: Module 18, Lesson 3 complete (84/90+ lessons total)
