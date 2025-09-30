# Async Patterns

## Concurrent Processing

```rust
use tokio::task::JoinSet;

/// Process multiple posts concurrently with error handling
pub async fn process_posts_concurrently(
    posts: Vec<Post>,
) -> BlogResult<Vec<ProcessedPost>> {
    let mut tasks = JoinSet::new();
    
    // Spawn concurrent tasks
    for post in posts {
        let task = tokio::spawn(async move {
            process_single_post(post).await
        });
        tasks.spawn(task);
    }
    
    let mut results = Vec::new();
    
    // Collect results with error handling
    while let Some(task_result) = tasks.join_next().await {
        match task_result {
            Ok(Ok(processed)) => results.push(processed),
            Ok(Err(e)) => log::error!("Failed to process post: {}", e),
            Err(e) => log::error!("Task panicked: {}", e),
        }
    }
    
    Ok(results)
}
```

## Resource Management

```rust
use tokio::sync::Semaphore;
use std::sync::Arc;

/// Database connection pool with concurrency limits
pub struct DatabasePool {
    pool: Arc<SurrealDB>,
    semaphore: Arc<Semaphore>,
}

impl DatabasePool {
    pub fn new(db: SurrealDB, max_connections: usize) -> Self {
        Self {
            pool: Arc::new(db),
            semaphore: Arc::new(Semaphore::new(max_connections)),
        }
    }
    
    pub async fn execute_query<F, T>(&self, operation: F) -> DbResult<T>
    where
        F: FnOnce(&SurrealDB) -> DbResult<T> + Send + 'static,
        T: Send + 'static,
    {
        let _permit = self.semaphore.acquire().await?;
        let pool = self.pool.clone();
        
        tokio::spawn(async move {
            operation(&pool).await
        })
        .await
        .map_err(|_| BlogError::Internal("Task cancelled".to_string()))?
    }
}
```

### Async Best Practices

**Proper Error Handling**
Handle async errors correctly:

```rust
pub async fn fetch_data() -> Result<Data, Error> {
    let client = HttpClient::new();
    
    // Use ? operator for early returns
    let response = client.get("https://api.example.com/data").await?;
    
    // Handle potential parsing errors
    let data = response.json::<Data>().await?;
    
    Ok(data)
}
```

**Cancellation Awareness**
Write cancellation-aware async code:

```rust
pub async fn process_stream<S, T>(mut stream: S) -> Result<Vec<T>, Error>
where
    S: Stream<Item = T> + Unpin,
{
    let mut results = Vec::new();
    
    loop {
        tokio::select! {
            // Process next item from stream
            Some(item) = stream.next() => {
                results.push(item);
            }
            // Handle cancellation
            _ = tokio::signal::ctrl_c() => {
                println!("Received cancellation signal");
                break;
            }
            // Timeout
            _ = tokio::time::sleep(Duration::from_secs(30)) => {
                println!("Operation timed out");
                break;
            }
        }
    }
    
    Ok(results)
}
```

**Backpressure Management**
Implement backpressure for high-throughput systems:

```rust
use tokio::sync::mpsc;

pub struct Processor {
    input: mpsc::Sender<WorkItem>,
    output: mpsc::Receiver<ProcessedItem>,
}

impl Processor {
    pub fn new(buffer_size: usize) -> Self {
        let (input_tx, mut input_rx) = mpsc::channel(buffer_size);
        let (output_tx, output_rx) = mpsc::channel(buffer_size);
        
        // Spawn processing task
        tokio::spawn(async move {
            while let Some(item) = input_rx.recv().await {
                let processed = process_item(item).await;
                
                // Handle backpressure - if output channel is full,
                // this will wait until there's space
                if let Err(_) = output_tx.send(processed).await {
                    break; // Output channel closed
                }
            }
        });
        
        Self {
            input: input_tx,
            output: output_rx,
        }
    }
    
    pub async fn submit(&self, item: WorkItem) -> Result<(), Error> {
        // This will return an error if the input buffer is full
        self.input.send(item).await
            .map_err(|_| Error::BufferFull)
    }
    
    pub async fn get_result(&mut self) -> Option<ProcessedItem> {
        self.output.recv().await
    }
}
```

### Async Patterns

**Async Iterator Pattern**
Create async iterators for data streams:

```rust
use futures::Stream;

pub struct AsyncFileReader {
    file: tokio::fs::File,
    buffer_size: usize,
}

impl AsyncFileReader {
    pub fn new(path: &str, buffer_size: usize) -> Self {
        Self {
            file: tokio::fs::File::open(path).await.unwrap(),
            buffer_size,
        }
    }
}

impl Stream for AsyncFileReader {
    type Item = Result<Vec<u8>, std::io::Error>;
    
    fn poll_next(
        mut self: std::pin::Pin<&mut Self>,
        cx: &mut std::task::Context<'_>,
    ) -> std::task::Poll<Option<Self::Item>> {
        let mut buffer = vec![0u8; self.buffer_size];
        
        match std::pin::Pin::new(&mut self.file).poll_read(cx, &mut buffer) {
            std::task::Poll::Ready(Ok(0)) => {
                // End of file
                std::task::Poll::Ready(None)
            }
            std::task::Poll::Ready(Ok(n)) => {
                buffer.truncate(n);
                std::task::Poll::Ready(Some(Ok(buffer)))
            }
            std::task::Poll::Ready(Err(e)) => {
                std::task::Poll::Ready(Some(Err(e)))
            }
            std::task::Poll::Pending => {
                std::task::Poll::Pending
            }
        }
    }
}
```

**Async Builder Pattern**
Use builder pattern for async configuration:

```rust
pub struct AsyncClientBuilder {
    timeout: Duration,
    retry_count: usize,
    buffer_size: usize,
}

impl AsyncClientBuilder {
    pub fn new() -> Self {
        Self {
            timeout: Duration::from_secs(30),
            retry_count: 3,
            buffer_size: 1024,
        }
    }
    
    pub fn timeout(mut self, timeout: Duration) -> Self {
        self.timeout = timeout;
        self
    }
    
    pub fn retry_count(mut self, retry_count: usize) -> Self {
        self.retry_count = retry_count;
        self
    }
    
    pub fn buffer_size(mut self, buffer_size: usize) -> Self {
        self.buffer_size = buffer_size;
        self
    }
    
    pub async fn build(self) -> Result<AsyncClient, Error> {
        let connection = establish_connection().await?;
        
        Ok(AsyncClient {
            connection,
            timeout: self.timeout,
            retry_count: self.retry_count,
            buffer_size: self.buffer_size,
        })
    }
}

pub struct AsyncClient {
    connection: Connection,
    timeout: Duration,
    retry_count: usize,
    buffer_size: usize,
}

impl AsyncClient {
    pub async fn request(&self, data: &[u8]) -> Result<Response, Error> {
        // Use configured timeout
        tokio::time::timeout(self.timeout, async {
            self.execute_request(data).await
        })
        .await
        .map_err(|_| Error::Timeout)?
    }
}
```

**Async Middleware Pattern**
Implement middleware for async request processing:

```rust
pub trait AsyncMiddleware {
    async fn handle(
        &self,
        request: Request,
        next: Next,
    ) -> Result<Response, Error>;
}

pub struct LoggingMiddleware;

#[async_trait::async_trait]
impl AsyncMiddleware for LoggingMiddleware {
    async fn handle(
        &self,
        request: Request,
        next: Next,
    ) -> Result<Response, Error> {
        let start = std::time::Instant::now();
        
        log::info!("Starting request: {:?}", request);
        
        let result = next.call(request).await;
        
        let duration = start.elapsed();
        
        match &result {
            Ok(response) => {
                log::info!("Request completed in {:?}: {:?}", duration, response.status);
            }
            Err(error) => {
                log::error!("Request failed in {:?}: {:?}", duration, error);
            }
        }
        
        result
    }
}

pub struct AuthMiddleware {
    api_key: String,
}

#[async_trait::async_trait]
impl AsyncMiddleware for AuthMiddleware {
    async fn handle(
        &self,
        request: Request,
        next: Next,
    ) -> Result<Response, Error> {
        if let Some(key) = request.headers.get("X-API-Key") {
            if key == self.api_key {
                return next.call(request).await;
            }
        }
        
        Err(Error::Unauthorized)
    }
}

pub struct AsyncApp {
    middlewares: Vec<Box<dyn AsyncMiddleware>>,
}

impl AsyncApp {
    pub fn new() -> Self {
        Self {
            middlewares: Vec::new(),
        }
    }
    
    pub fn add_middleware<M: AsyncMiddleware + 'static>(mut self, middleware: M) -> Self {
        self.middlewares.push(Box::new(middleware));
        self
    }
    
    pub async fn handle_request(&self, request: Request) -> Result<Response, Error> {
        let mut next = Next::new(|req| self.process_request(req));
        
        // Apply middlewares in reverse order
        for middleware in self.middlewares.iter().rev() {
            let current_next = next;
            next = Next::new(move |req| {
                let middleware = middleware.clone();
                let next = current_next.clone();
                async move { middleware.handle(req, next).await }
            });
        }
        
        next.call(request).await
    }
    
    async fn process_request(&self, request: Request) -> Result<Response, Error> {
        // Actual request processing logic
        Ok(Response::new("Hello, World!"))
    }
}
```

### Async Testing

**Testing Async Code**
Use proper async testing:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tokio::time::{sleep, Duration};

    #[tokio::test]
    async fn test_async_function() {
        let result = async_function().await;
        assert!(result.is_ok());
    }

    #[tokio::test]
    async fn test_concurrent_processing() {
        let items = vec![1, 2, 3, 4, 5];
        let results = process_concurrently(items).await;
        
        assert_eq!(results.len(), 5);
        assert!(results.iter().all(|r| r.is_ok()));
    }

    #[tokio::test]
    async fn test_timeout_handling() {
        let result = tokio::time::timeout(
            Duration::from_millis(100),
            slow_operation(),
        ).await;
        
        assert!(matches!(result, Err(_))); // Should timeout
    }

    #[tokio::test]
    async fn test_error_handling() {
        let result = failing_operation().await;
        assert!(matches!(result, Err(Error::OperationFailed)));
    }
}
```

**Mocking Async Dependencies**
Use mocking for async dependencies:

```rust
#[cfg(test)]
mockall::mock! {
    pub AsyncDatabase {}
    
    #[async_trait::async_trait]
    impl DatabaseTrait for AsyncDatabase {
        async fn save(&self, data: &Data) -> Result<(), Error>;
        async fn get(&self, id: &str) -> Result<Data, Error>;
    }
}

#[tokio::test]
async fn test_service_with_mock() {
    let mut mock_db = MockAsyncDatabase::new();
    
    // Setup mock expectations
    mock_db.expect_save()
        .times(1)
        .returning(|_| Ok(()));
    
    mock_db.expect_get()
        .times(1)
        .returning(|_| Ok(Data::new("test")));
    
    let service = Service::new(mock_db);
    let result = service.process_data("test").await;
    
    assert!(result.is_ok());
}
```

These patterns have worked well in production Rust environments and should help you build robust, performant systems.

Next: [Python Engineering Best Practices](./python-practices.md)