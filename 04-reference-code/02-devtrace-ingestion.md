# Reference: High-Throughput Ingestion Buffer

> **Concept Proven:** "Zero-copy ingestion over Tokio MPSC channels with a bounded 10k-event buffer." (ADR-002)
> **Source Project:** DevTrace (`DevTrace/logger`)

This proves how a high-velocity event stream is safely handled without creating severe backpressure on the ingestion API. By dropping the payload onto a bounded Tokio MPSC channel, the HTTP listener immediately returns a success code, while a background worker safely commits the batches to SQLite.

## The Store & Background Worker
Notice how the initialization spawns an independent Tokio thread that loops over `rx.recv()`. If the 10,000-event buffer fills up, the `try_send` in the `add` method drops the log, preventing an out-of-memory crash.

**`src/logger/store.rs`**
```rust
use tokio::sync::mpsc;
use sqlx::sqlite::SqlitePool;

pub struct LogStore {
    tx: mpsc::Sender<RequestLog>,
    pool: SqlitePool,
}

impl LogStore {
    pub async fn new() -> Self {
        // ... SQLite connection pool setup ...

        // 1. Create the bounded 10k buffer
        let (tx, mut rx) = mpsc::channel::<RequestLog>(10_000);
        let worker_pool = pool.clone();

        // 2. Spawn the background worker
        tokio::spawn(async move {
            println!("👷 Background Event Worker started. Waiting for logs...");

            // 3. Drain the channel asynchronously
            while let Some(log) = rx.recv().await {
                let method_str = format!("{:?}", log.request.method);

                let result = sqlx::query(
                    "INSERT INTO logs (method, path, status, duration_ms, start_time, end_time)
                     VALUES (?, ?, ?, ?, ?, ?)"
                )
                .bind(&method_str)
                .bind(&log.request.path)
                // ... bindings ...
                .execute(&worker_pool)
                .await;

                match result {
                    Ok(_) => {
                        // Silently succeed — the belt keeps moving
                    }
                    Err(e) => {
                        eprintln!("❌ Database Write Error: {}", e);
                    }
                }
            }
        });

        Self { tx, pool }
    }

    /// Drops the log onto the conveyor belt without blocking.
    pub fn add(&self, log: RequestLog) {
        if let Err(e) = self.tx.try_send(log) {
            eprintln!("⚠️  Conveyor belt full! Dropping log: {}", e);
        }
    }
}
```
